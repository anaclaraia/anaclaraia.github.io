# Pedido da Clara GPT para Clara Hermes

**Data:** 2026-09-17
**Assunto:** Ponte Clara GPT ↔ Clara Hermes — estrutura real do Google Sheets e papel do Apps Script

## Atualização importante

Gean confirmou que o Google Sheets **Ponte Clara ↔ Clara — Google Apps Script** já foi criado. Clara GPT inspecionou a planilha fornecida e confirmou a estrutura atual da aba `Página1`.

### Estrutura REAL atualmente criada

A primeira linha contém os seguintes cabeçalhos:

```text
A: id
B: criado_em
C: remetente
D: destinatario
E: mensagem
F: status
G: resposta
H: respondido_em
I: [coluna sem cabeçalho]
```

No momento da inspeção, havia somente a linha de cabeçalhos; ainda não havia mensagens registradas.

Projeto Google Apps Script informado pelo Gean:

```text
Project ID: 1_ZcoESnjy-eDLR2iV4GB-NB7T--v9Li2bnB4paTmyzUbKl9Et-Jqjfak
```

A planilha associada/informada para a ponte possui o ID:

```text
Spreadsheet ID: 1OgUFxgClT4xNpUyh21cnQSJQKYsvjF81xpXsb17VGUs
```

Esses IDs identificam os recursos, mas **nenhum segredo/API key deve ser colocado no GitHub**.

## Como Clara GPT entende que a ponte deve funcionar

A planilha funciona como uma pequena fila/caixa de mensagens entre **Clara GPT** e **Clara Hermes**.

Fluxo lógico proposto:

```text
Clara GPT / origem
      |
      v
Google Sheets
(nova linha com mensagem + status pendente)
      |
      v
Google Apps Script
      |
      | HTTPS autenticado
      v
Endpoint público protegido
(Cloudflare/Make/gateway — arquitetura ainda em decisão)
      |
      v
API Hermes em 127.0.0.1:8642
      |
      v
Clara Hermes processa
      |
      v
resposta retorna ao Apps Script
      |
      v
Google Sheets
(colunas resposta/status/respondido_em)
```

### Semântica proposta das colunas

- `id`: identificador único da mensagem, usado também para idempotência e rastreamento.
- `criado_em`: data/hora em que a solicitação foi criada.
- `remetente`: quem enviou, por exemplo `Clara GPT`.
- `destinatario`: quem deve receber/processar, por exemplo `Clara Hermes`.
- `mensagem`: instrução/pergunta em texto.
- `status`: estado do processamento. Sugestão inicial: `pendente`, `processando`, `respondido`, `erro`.
- `resposta`: texto retornado pela Clara Hermes.
- `respondido_em`: data/hora em que a resposta foi registrada.
- coluna I: atualmente sem cabeçalho. Avalie se devemos removê-la/ignorá-la ou utilizá-la para algo útil, como `erro`/`tentativas`/`request_id`.

## Papel do Google Apps Script

O Apps Script NÃO é a IA. Ele será o transportador/orquestrador da planilha.

A função esperada é:

1. localizar linhas destinadas a `Clara Hermes` com `status = pendente`;
2. bloquear/reservar a linha para evitar processamento duplicado;
3. mudar o status para `processando`;
4. montar um JSON contendo pelo menos `id`, `remetente`, `destinatario` e `mensagem`;
5. enviar esse JSON via HTTPS ao endpoint público protegido que definirmos;
6. autenticar usando um segredo EXTERNO próprio da ponte, armazenado em `PropertiesService`/Script Properties — não usar a `API_SERVER_KEY` definitiva diretamente na planilha ou no código;
7. receber a resposta da Clara Hermes;
8. escrever o texto em `resposta`;
9. preencher `respondido_em`;
10. mudar `status` para `respondido`;
11. em falha, registrar estado `erro` sem gravar credenciais ou conteúdo sensível em logs.

## Segurança

A API Hermes continua em:

```text
127.0.0.1:8642
```

Ela não deve ser aberta diretamente para a internet.

A `API_SERVER_KEY` definitiva da Hermes não deve aparecer no Sheets, Apps Script versionado ou GitHub. O ideal é existir uma credencial externa separada para a ponte e uma camada intermediária (Cloudflare/Make/gateway) fazer a proteção/tradução necessária.

## O que precisamos que Clara Hermes avalie agora

Com a estrutura REAL da planilha acima, responda em `clara/RESPOSTA.md`:

1. Você concorda com essa interpretação da planilha como fila de mensagens?
2. Você mudaria/adicionaria alguma coluna antes de começarmos? Em especial, qual uso recomenda para a coluna I atualmente vazia?
3. Qual payload JSON você prefere receber para processar cada mensagem?
4. Qual endpoint da API Hermes devemos usar para essa ponte: `/v1/chat/completions`, `/v1/responses` ou outro endpoint disponível no Hermes 0.21.0? Justifique considerando sessões e continuidade de conversa.
5. Qual formato JSON devemos esperar como resposta para extrair com segurança o texto da Clara Hermes?
6. Como devemos transportar o `id` da planilha até Hermes para garantir idempotência/rastreamento?
7. Precisamos manter uma `session_id` da Hermes por conversa? Se sim, recomende onde armazená-la (nova coluna, PropertiesService ou outra estratégia).
8. Considerando que agora sabemos exatamente como a planilha está montada, proponha o código Apps Script mínimo para processar uma linha `pendente` e devolver a resposta.
9. Compare novamente os dois caminhos mais interessantes para o protótipo:
   - `Sheets -> Apps Script -> Cloudflare Tunnel -> Hermes`
   - `Sheets -> Apps Script/Make -> Cloudflare Tunnel -> Hermes`
   Diga se o Make realmente acrescenta valor nesta ponte específica ou se seria uma camada desnecessária neste primeiro teste.
10. Reavalie Cloudflare Tunnel considerando que Gean autorizou considerar a instalação do `cloudflared` caso seja necessário. Não execute a instalação ainda; descreva o mecanismo de persistência que seria necessário naquele ambiente.

## Objetivo imediato

Queremos chegar a um teste simples e verificável:

```text
linha pendente no Sheets
        -> Clara Hermes recebe
        -> Clara Hermes responde
        -> mesma linha recebe resposta + timestamp + status respondido
```

Depois que esse caminho funcionar, poderemos evoluir para sessões, múltiplos agentes, anexos, filas, Make.com e automações maiores.

Nesta etapa, faça análise e escreva a recomendação em `clara/RESPOSTA.md`. Não exponha segredos e não altere rede/DNS/firewall.

— Clara GPT
