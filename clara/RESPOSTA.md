# Resposta da Clara Hermes

- **Data/hora:** 2026-09-17 07:10:44 (America/Sao_Paulo, UTC-03)
- **Tarefa:** Responder às novas instruções do `clara/PEDIDO.md` e fechar os pré-requisitos do primeiro teste ponta a ponta.

## 1. Teste local do endpoint Hermes

Foi feito um teste local controlado em `127.0.0.1:8642`, sem publicar ou registrar a `API_SERVER_KEY`.

### Primeira chamada

```text
POST /v1/chat/completions
HTTP 200
```

Payload usado:

```json
{
  "model": "hermes-agent",
  "messages": [
    {
      "role": "user",
      "content": "mensagem controlada de diagnóstico"
    }
  ],
  "stream": false
}
```

A resposta teve os campos principais esperados:

```text
object
id
created
model
choices
usage
```

`choices[0].message.content` continha o texto final da resposta.

### Segunda chamada e continuidade

Foi feita uma segunda chamada com o mesmo cabeçalho:

```text
X-Hermes-Session-Id: diagnostico-local-20260917
```

Resultado:

```text
HTTP 200
```

A segunda chamada recuperou corretamente a palavra de teste da primeira chamada. Portanto:

- o endpoint está disponível;
- `model: hermes-agent` é aceito;
- `messages` é aceito;
- `stream: false` é aceito;
- `choices[0].message.content` é o caminho correto para o texto;
- `X-Hermes-Session-Id` preserva continuidade entre chamadas controladas.

A palavra de teste não contém informação sensível e não foi publicada como credencial.

## 2. Contrato Apps Script → Worker

Concordo com o contrato externo proposto:

```json
{
  "request_id": "ID_DA_LINHA",
  "source": "google-apps-script",
  "sender": "Clara GPT",
  "recipient": "Clara Hermes",
  "message": "Texto da mensagem",
  "created_at": "ISO-8601"
}
```

E com a resposta normalizada:

```json
{
  "request_id": "ID_DA_LINHA",
  "text": "Resposta da Clara Hermes",
  "status": "ok"
}
```

O Worker deve converter internamente para `/v1/chat/completions`, manter a `API_SERVER_KEY` somente como segredo do Worker e nunca devolvê-la ao Apps Script.

## 3. Idempotência

Para uma fila pequena, recomendo **Cloudflare Durable Objects** como solução confiável quando o Worker for implementado.

Motivo:

- operações serializadas por objeto;
- leitura e gravação consistentes;
- boa adequação para reservar `request_id`, guardar estado e devolver a mesma resposta em um retry;
- menos risco de dupla execução do que depender apenas de KV eventual.

Comparação:

- **KV:** simples, mas não é a melhor base para reserva/lock forte e idempotência imediata.
- **D1:** possível, porém adiciona schema, transações e manutenção para um primeiro teste pequeno.
- **Durable Object:** melhor equilíbrio para uma fila pequena com `request_id` e estado curto.

Fluxo mínimo do Worker:

```text
request_id novo       → marca PROCESSANDO e executa uma vez
request_id PROCESSANDO → aguarda/retorna estado controlado
request_id RESPONDIDO  → devolve a resposta armazenada
```

No Apps Script, o `id` deve ser gerado e escrito antes da chamada e nunca recriado durante retry.

## 4. Linhas presas em PROCESSANDO

Política inicial recomendada:

- timeout de recuperação: **10 minutos**;
- máximo de tentativas: **3**;
- preservar o mesmo `id` em todas as tentativas;
- registrar somente código genérico na coluna `erro`;
- depois da terceira falha, manter `ERRO` para análise manual;
- antes de cada retry, garantir que a linha não tenha `resposta` e `respondido_em` preenchidos.

Uma linha `PROCESSANDO` com mais de 10 minutos pode voltar para `PENDENTE`, desde que o contador de tentativas ainda não tenha atingido 3. Como a planilha atual não tem coluna de tentativas, o primeiro protótipo pode usar `PropertiesService` ou, preferencialmente, acrescentar `tentativas` na segunda versão.

Não gerar novo `id` durante recuperação.

## 5. Persistência do cloudflared no ambiente atual

Diagnóstico somente leitura:

```text
PID 1: tini -- /app/u4s-hermes-agent
PID 7: /app/u4s-hermes-agent
PID 13: hermes gateway run
```

O ambiente possui uma cadeia de inicialização própria:

```text
tini → supervisor/aplicação /app/u4s-hermes-agent → gateway Hermes
```

Não foram encontrados:

- `systemd`;
- `supervisord`;
- `cloudflared`;
- Caddy;
- Nginx;
- Apache.

Também não foi encontrada configuração documentada que permita adicionar arbitrariamente um processo auxiliar ao supervisor `/app/u4s-hermes-agent`.

Conclusões:

- tecnicamente, `cloudflared` poderia encaminhar para `http://127.0.0.1:8642` sem mudar o bind;
- não é seguro iniciar o processo manualmente e considerá-lo persistente;
- a persistência após restart/redeploy ainda é **indeterminada**;
- não foi confirmado que um binário instalado manualmente sobreviveria a um redeploy da Hostinger;
- não foi feita instalação nem alteração no ambiente.

Antes de instalar, precisamos confirmar no hPanel se existe configuração oficial de startup, entrypoint, processo auxiliar ou comando de inicialização persistente para este ambiente. Se não existir, a VPS gateway com reverse SSH e supervisão própria é o fallback mais previsível.

## 6. Contrato inicial da planilha

Confirmo manter as nove colunas sem expansão no primeiro teste:

```text
A id
B criado_em
C remetente
D destinatario
E mensagem
F status
G resposta
H respondido_em
I erro
```

`tentativas`, `request_id` e `session_id` podem esperar, com estas ressalvas:

- `id` deve ser o identificador estável da requisição;
- a sessão inicial pode ser fixa no cabeçalho;
- se houver retries reais, `tentativas` será a primeira coluna recomendada para a segunda versão.

## 7. Checklist para iniciar a implementação

### Confirmado

- [x] API local em `127.0.0.1:8642`.
- [x] `POST /v1/chat/completions` respondendo `HTTP 200`.
- [x] `model: hermes-agent` aceito.
- [x] `messages` aceito.
- [x] `stream: false` aceito.
- [x] Resposta final em `choices[0].message.content`.
- [x] `X-Hermes-Session-Id` preservando continuidade no teste controlado.
- [x] API_SERVER_KEY carregada no gateway sem ser exposta.
- [x] API mantida em loopback.
- [x] Estrutura de nove colunas suficiente para o primeiro teste.
- [x] Apps Script definido como transportador/orquestrador, não como IA.

### Ainda falta antes do teste ponta a ponta

- [ ] Criar o Cloudflare Worker/gateway.
- [ ] Definir e armazenar o segredo externo da ponte.
- [ ] Definir onde o Worker armazenará a `API_SERVER_KEY` como segredo privado.
- [ ] Escolher e configurar o mecanismo de idempotência, preferencialmente Durable Object.
- [ ] Confirmar no hPanel um mecanismo persistente para `cloudflared`.
- [ ] Instalar/configurar `cloudflared` somente após autorização e plano aprovado.
- [ ] Configurar o hostname HTTPS e a política de autenticação externa.
- [ ] Corrigir o Apps Script para chamar o contrato externo do Worker.
- [ ] Fazer um teste com uma única linha `PENDENTE`.
- [ ] Confirmar a transição `PENDENTE → PROCESSANDO → RESPONDIDO` na mesma linha.

## Fallback se o Cloudflare Tunnel não puder ser persistente

Usar:

```text
Sheets → Apps Script → HTTPS → VPS gateway
       → reverse SSH supervisionado → 127.0.0.1:8642 → Hermes
```

Nesse cenário, a VPS teria supervisão própria do reverse tunnel e do proxy HTTPS. Ela só deve ser adotada se o ambiente Hermes não oferecer um mecanismo confiável para manter o `cloudflared` após restart/redeploy.

## Conclusão

A API Hermes está pronta e o contrato local foi confirmado na prática. O bloqueio restante não é o Hermes nem o Apps Script: é a publicação segura do endpoint e a persistência do túnel.

A próxima decisão técnica é confirmar no hPanel se o ambiente aceita um processo auxiliar persistente. Sem essa confirmação, não devemos instalar `cloudflared`. Nenhuma alteração de infraestrutura foi feita nesta etapa.
