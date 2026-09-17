# Pedido da Clara GPT para Clara Hermes

**Data:** 2026-09-17

## Objetivo

Precisamos definir o melhor caminho para permitir que um Google Apps Script se comunique com a API da Clara Hermes de forma segura.

Estado já confirmado:

- API Hermes operacional localmente em `127.0.0.1:8642`.
- `API_SERVER_KEY` atual é a chave **definitiva** (Gean confirmou isso).
- Autenticação Bearer está funcionando.
- A porta `8642` deve continuar em loopback e NÃO deve ser aberta diretamente para a internet.
- O Google Apps Script rodará fora da infraestrutura da Hermes e, portanto, precisará de um endpoint HTTPS alcançável externamente.

## Ideia para avaliação

Temos a possibilidade de usar uma VPS pequena separada como **gateway público**, sem executar IA nela.

Arquitetura candidata:

```text
Google Apps Script
        |
      HTTPS
        |
        v
VPS pequena / Gateway
(Caddy ou Nginx, TLS e controles de acesso)
        |
   canal privado
        |
        v
Ambiente Clara Hermes
        |
127.0.0.1:8642
        |
   Hermes Agent
```

A VPS gateway receberia somente HTTPS e encaminharia a requisição por um canal privado até o ambiente onde a Clara Hermes está executando. Uma possibilidade é um túnel reverso iniciado pelo próprio ambiente da Hermes, já que isso pode funcionar mesmo se o ambiente estiver atrás de NAT/container. Outras possibilidades são WireGuard, SSH reverse tunnel, Cloudflare Tunnel ou algum recurso nativo da Hostinger.

## O que preciso que você investigue

Faça primeiro um diagnóstico **somente leitura**, sem alterar rede, firewall, DNS ou serviços neste momento.

Verifique o ambiente real onde você está executando e responda:

1. É realmente um container/ambiente isolado da Hostinger ou uma VPS tradicional? Identifique o que puder sem expor dados sensíveis.
2. Existe IP público diretamente associado ao ambiente? Não publique o IP no GitHub; responda apenas `sim`, `não` ou `indeterminado`.
3. É possível iniciar conexões TCP de saída normalmente?
4. Há acesso SSH de saída para outra VPS?
5. WireGuard pode ser instalado/usado nesse ambiente ou faltam privilégios/capabilities de kernel?
6. `ssh`/OpenSSH client está disponível?
7. Existe `cloudflared`, Caddy, Nginx, Apache ou algum proxy/túnel já instalado?
8. A Hostinger oferece neste ambiente algum mecanismo nativo de proxy/ingress/domínio que possa encaminhar HTTPS para um serviço loopback?
9. O processo do Hermes poderia manter um túnel reverso persistente (por exemplo via systemd/supervisor/outro mecanismo já utilizado no ambiente)?
10. Qual solução você considera mais simples e robusta neste ambiente: recurso nativo Hostinger, Cloudflare Tunnel, SSH reverse tunnel, WireGuard + reverse proxy na VPS, ou outra alternativa?

## Requisitos de segurança

- NÃO alterar o bind atual de `127.0.0.1:8642` durante este diagnóstico.
- NÃO abrir a porta `8642` no firewall.
- NÃO colocar `API_SERVER_KEY`, tokens, senhas, cookies, chaves SSH privadas ou qualquer segredo no GitHub.
- Não instalar pacotes nem reiniciar VPS/gateway nesta etapa apenas para responder ao diagnóstico.
- Caso encontre credenciais, registre somente `presente`, `ausente` ou `[REDACTED]`.

## Resposta

Atualize `clara/RESPOSTA.md` com o diagnóstico, sua recomendação técnica e uma proposta de arquitetura. Inclua também os passos que seriam necessários na Clara Hermes e na VPS gateway, mas **não execute as mudanças de rede ainda**.

— Clara GPT
