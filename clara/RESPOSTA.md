# Resposta da Clara Hermes

- **Data/hora:** 2026-09-17 06:30:09 (America/Sao_Paulo, UTC-03)
- **Tarefa:** Diagnosticar, somente com leitura, o melhor caminho para permitir que um Google Apps Script acesse a API Hermes com segurança.

## Diagnóstico do ambiente

1. **Tipo de ambiente:** não foi possível confirmar, somente pelos sinais locais, se é uma VPS tradicional ou um ambiente gerenciado da Hostinger. O processo principal é iniciado por `tini` com `/app/u4s-hermes-agent`, indicando uma camada de supervisão/empacotamento. Não foram encontrados marcadores conclusivos de Docker/Kubernetes no cgroup.
2. **IP público diretamente associado:** **indeterminado**. Nenhum IP foi publicado nem incluído neste arquivo.
3. **Conexões TCP de saída:** **sim**. Foram testadas conexões de saída para HTTPS (porta 443) e SSH (porta 22).
4. **SSH de saída:** o cliente consegue alcançar a porta SSH externamente. A existência de credenciais e o acesso a uma VPS de destino ainda não foram testados.
5. **WireGuard:** não está disponível como ferramenta (`wg`/`wg-quick` ausentes), o módulo não foi encontrado e o processo não possui `CAP_NET_ADMIN`. Não é uma opção viável neste ambiente sem privilégios e instalação adicional.
6. **Cliente OpenSSH:** disponível (`ssh` e `ssh-keygen`).
7. **Proxy/túnel instalado:** não foram encontrados `cloudflared`, Caddy, Nginx, Apache, `httpd` ou `supervisord`. Também não há `systemctl` disponível.
8. **Recurso nativo de proxy/ingress da Hostinger:** **indeterminado** a partir do ambiente local. Não foi feita alteração no hPanel nem na rede.
9. **Túnel reverso persistente:** conexões de saída são possíveis, mas não há atualmente um gerenciador local comum (`systemd`, Supervisor ou `autossh`) confirmado para manter o túnel. Seria necessário usar o mecanismo de execução já fornecido pelo ambiente ou uma VPS gateway com supervisão própria.

## Estado atual confirmado

- A API Hermes continua vinculada somente a `127.0.0.1:8642`.
- A API local respondeu com sucesso aos endpoints testados anteriormente.
- A porta da API não foi aberta diretamente para a internet.
- Nenhum firewall, DNS, proxy, túnel ou serviço foi alterado durante este diagnóstico.
- Nenhuma chave, senha, token, IP ou credencial foi incluído neste arquivo.

## Recomendação técnica

### Opção recomendada: gateway público separado + túnel reverso SSH

Para este ambiente, a alternativa mais simples de validar é:

```text
Google Apps Script
        |
      HTTPS
        |
VPS gateway pública
(reverse proxy + TLS + rate limit)
        |
SSH reverse tunnel iniciado pelo ambiente Hermes
        |
127.0.0.1:8642
```

Motivos:

- O SSH client já está disponível.
- A saída TCP para a porta 22 foi confirmada.
- A API Hermes permanece protegida em loopback.
- A VPS gateway pode cuidar do domínio, TLS, autenticação externa, limitação de requisições e logs.
- A chave interna da API pode permanecer somente no gateway/túnel, sem ser colocada no Google Apps Script ou no GitHub.

A persistência do túnel ainda precisa ser projetada de acordo com o mecanismo de execução da Hostinger. Não deve ser implementada com um processo solto sem supervisão.

### Alternativa: Cloudflare Tunnel

Pode ser mais simples operacionalmente se o domínio estiver no Cloudflare e o ambiente permitir executar `cloudflared` de forma persistente. Porém, `cloudflared` não está instalado neste ambiente e essa alternativa exigiria instalação/configuração posterior, além de um mecanismo de inicialização e atualização segura.

### Opções não recomendadas neste momento

- Abrir diretamente a porta `8642`: rejeitado por segurança.
- WireGuard neste ambiente: não há ferramenta nem capacidade de kernel necessária.
- Instalar Nginx/Caddy aqui sem confirmar o modelo de execução: prematuro.
- Expor a `API_SERVER_KEY` no Apps Script: não recomendado; deve existir uma camada de autenticação externa separada.

## Arquitetura de segurança proposta

- Manter `127.0.0.1:8642` fechado para a internet.
- Usar HTTPS obrigatório no gateway público.
- Usar um segredo de entrada separado para o Google Apps Script, armazenado no `PropertiesService`, nunca no código público ou no GitHub.
- Fazer o gateway traduzir a autenticação externa para a autenticação interna sem registrar valores sensíveis.
- Aplicar rate limit, limite de tamanho de requisição, timeout e logs sem payloads ou credenciais.
- Permitir somente os endpoints necessários ao Apps Script, em vez de publicar toda a API administrativa.
- Validar primeiro com um endpoint de saúde não sensível e uma chamada controlada, antes de liberar chat completo.

## Conclusão

O melhor caminho inicial é **VPS gateway pública com HTTPS e SSH reverse tunnel**, mantendo a API Hermes em loopback. Cloudflare Tunnel é uma alternativa potencialmente mais simples se o domínio e a execução persistente de `cloudflared` estiverem disponíveis. Nenhuma mudança de infraestrutura foi executada; o próximo passo deve ser escolher uma dessas alternativas e então fazer um plano de implementação com backup, autenticação separada e validação ponta a ponta.
