# Relatório consolidado — configuração da ponte Hermes/Cloudflare

- **Data:** 2026-09-17
- **Período:** aproximadamente 06:00 em diante, America/Sao_Paulo (UTC-03)
- **Objetivo:** preparar e testar temporariamente a exposição HTTPS da API Hermes para a futura ponte Google Apps Script/Sheets → Cloudflare → Hermes.
- **Repositório:** `anaclaraia/anaclaraia.github.io`
- **Segredos:** valores de tokens, chaves OAuth e credenciais foram omitidos e registrados apenas como `[REDACTED]` quando necessário.

## 1. Resultado executivo

A API Hermes foi configurada e validada localmente em `127.0.0.1:8642`. O Tunnel `clara-hermes-api` foi criado/configurado no painel Cloudflare, o hostname `api.50x.com.br` passou a usar o Tunnel e o conector temporário `cloudflared` foi iniciado.

O primeiro bloqueio externo foi o Cloudflare Browser Integrity Check (BIC), que devolvia `403/1010` para chamadas automatizadas. A causa foi confirmada nos eventos e foi implantada uma regra limitada ao hostname da API para pular somente o BIC. As proteções relacionadas ao Listmonk na VPS nova da Locaweb permaneceram ativas.

Após a correção do BIC, o erro externo passou a ser `HTTP 503`. A causa foi confirmada nos logs do `cloudflared`: não havia regras de ingress carregadas no processo iniciado com o token, portanto o processo retornava `503` para todas as requisições. A API local continuava respondendo `HTTP 200`.

**Estado final deste relatório:** DNS e Tunnel alcançáveis; BIC resolvido; API local funcionando; endpoint externo ainda pendente de `HTTP 200` porque a configuração de ingress do Tunnel precisa ser carregada no mesmo Tunnel usado pelo conector.

## 2. API Hermes

Foi ajustado o ambiente do Hermes para disponibilizar o `api_server` somente no loopback:

```yaml
api_server:
  enabled: true
  host: 127.0.0.1
  port: 8642
```

Foi criado backup prévio da configuração. A ausência inicial de `API_SERVER_KEY` foi diagnosticada pela mensagem `API_SERVER_KEY is required`; depois a chave foi configurada no ambiente efetivo sem ser exposta e o gateway foi reiniciado uma vez, sem reiniciar a VPS inteira.

Testes locais confirmados:

- `GET http://127.0.0.1:8642/health` → `HTTP 200`;
- `POST /v1/chat/completions` → `HTTP 200`;
- `model: hermes-agent` aceito;
- `messages` aceito;
- `stream: false` aceito;
- resposta em `choices[0].message.content`;
- continuidade confirmada com `X-Hermes-Session-Id`.

A porta `8642` permaneceu privada, sem exposição direta à Internet. A porta `8643` existente foi identificada como porta de métricas, não como a API Hermes.

## 3. Google Workspace e contrato da ponte

Foi configurado o OAuth oficial do Google, com os arquivos locais protegidos e sem publicação de valores sensíveis. Foram localizados e atualizados os documentos de coordenação da Clara no Drive, e as gravações foram verificadas.

A Gmail API foi ativada e o acesso aos rascunhos foi confirmado. Foi criado um baseline de 17 rascunhos para considerar somente mensagens novas ou editadas. A rotina temporária de monitoramento foi configurada sem envio automático.

O contrato técnico definido para a futura ponte é:

```text
Google Sheets → Apps Script → endpoint HTTPS autenticado → Hermes
```

O Apps Script deverá chamar internamente `/v1/chat/completions`, mantendo a `API_SERVER_KEY` somente no lado protegido do gateway/Worker. A primeira prova ponta a ponta deverá usar uma única linha `PENDENTE` e confirmar `PENDENTE → PROCESSANDO → RESPONDIDO`.

Também foram documentadas idempotência por `request_id`, recuperação de linhas presas em `PROCESSANDO`, limite inicial de tentativas e uso futuro de Durable Objects ou mecanismo equivalente.

## 4. Cloudflare Tunnel e DNS

Foi escolhido o Tunnel existente:

```text
Nome: clara-hermes-api
ID: 1e2491c5-0545-44a5-8c76-56c8528c1152
```

A rota pública configurada no painel foi planejada para:

```text
api.50x.com.br → http://127.0.0.1:8642
```

O registro A de teste do hostname foi removido e substituído pelo CNAME do Tunnel, com proxy Cloudflare ativo:

```text
api.50x.com.br → 1e2491c5-0545-44a5-8c76-56c8528c1152.cfargotunnel.com
```

A zona `50x.com.br` permaneceu ativa, e os demais registros DNS foram preservados, incluindo os hostnames de aplicações/streaming e os registros MX, SPF, DKIM e DMARC. O DNS público passou a responder por endereços da Cloudflare.

## 5. cloudflared

O binário foi instalado em uma pasta gravável do ambiente atual:

```text
/data/workspace/bin/cloudflared
```

Características verificadas:

```text
Versão: 2026.9.1
Arquitetura: Linux amd64
Permissão: executável
```

A variável `CLOUDFLARE_TUNNEL_TOKEN` foi adicionada ao ambiente do Hermes. O valor nunca foi publicado neste relatório, no GitHub ou no chat.

O conector foi iniciado temporariamente com `cloudflared tunnel run --token`, registrou conexão QUIC e ficou ativo. A execução é provisória: não foi resolvida persistência após restart/redeploy no ambiente atual. A persistência definitiva será tratada na migração para a nova VPS.

## 6. BIC, WAF e eventos de segurança

O teste externo inicial retornou:

```text
HTTP 403
Cloudflare error 1010
```

A consulta dos eventos identificou exatamente:

```text
Host: api.50x.com.br
Source: bic
Rule: bic
User-Agent: Python-urllib/3.13
```

Foi implantada uma regra de segurança específica:

```text
Nome: Skip BIC for api.50x.com.br
Expressão: (http.host eq "api.50x.com.br")
Ação: Skip
Escopo: somente Browser Integrity Check
```

Não foram pulados Managed Rules, Custom Rules, Rate Limiting ou Bot Fight Mode. O BIC global não foi desativado.

Os eventos descritos como “WordPress RCE” pertencem ao Listmonk instalado na VPS nova da Locaweb. Eles não justificam desativar proteções globais e as regras relacionadas continuam ativas.

## 7. Diagnóstico final do HTTP 503

Após a implantação da exceção do BIC, foram executados testes externos reais:

```text
https://api.50x.com.br/health    → HTTP 503
https://api.50x.com.br/v1/models  → HTTP 503
```

Em paralelo, o teste local continuou retornando:

```text
http://127.0.0.1:8642/health → HTTP 200
```

O processo `cloudflared` permaneceu ativo, mas seus logs informaram:

```text
No ingress rules were defined in provided config (if any) nor from the cli,
cloudflared will return 503 for all incoming HTTP requests
```

Conclusão: o problema atual não é a API Hermes, DNS ou BIC. O processo `cloudflared` usado pelo conector está sem regra de ingress carregada. A configuração do mesmo Tunnel usado pelo token precisa conter uma rota para `http://127.0.0.1:8642` e uma regra final de fallback `http_status:404`, ou ser carregada corretamente pelo modo remoto correspondente.

## 8. Limitações e próximos passos

- Não configurar o Apps Script em produção enquanto `https://api.50x.com.br/health` não retornar `HTTP 200`.
- No painel Cloudflare, confirmar que a rota pertence ao Tunnel `clara-hermes-api` correto e que o serviço é `HTTP`, não `HTTPS`.
- Garantir que a configuração de ingress foi publicada/carregada no mesmo Tunnel cujo token está em uso.
- Repetir `/health` e `/v1/models` após a atualização e registrar o retorno.
- Depois do `HTTP 200`, criar o gateway/Worker autenticado, sem expor `API_SERVER_KEY` ao Apps Script.
- Fazer a primeira prova com uma única linha da planilha.
- Resolver a persistência do `cloudflared` somente na nova VPS, com mecanismo de supervisão confirmado.
- Não alterar as proteções do Listmonk nem registros DNS não relacionados.

## 9. Proteção de segredos

Não foram publicados neste relatório:

- `API_SERVER_KEY`;
- `CLOUDFLARE_TUNNEL_TOKEN`;
- tokens OAuth do Google;
- chaves privadas;
- senhas;
- dados pessoais sensíveis.

Os valores foram omitidos ou representados como `[REDACTED]`.

---

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
