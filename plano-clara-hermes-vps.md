# Plano de implantação da Clara autônoma na VPS

## Objetivo

Criar uma nova **Clara baseada no Hermes Agent**, funcionando 24 horas por dia em uma VPS Debian 12 e com autonomia real para trabalhar.

A Clara deverá poder:

- instalar e atualizar ferramentas;
- administrar serviços da VPS;
- usar terminal e sudo;
- configurar aplicações, Nginx, Docker e serviços necessários;
- possuir identidade e e-mail próprios;
- enviar e receber e-mails;
- administrar o DNS e os subdomínios do domínio destinado a ela;
- trabalhar com GitHub;
- publicar e manter o próprio site/blog pelo GitHub Pages;
- integrar novas ferramentas, APIs e MCPs quando forem necessárias;
- trabalhar sem depender de intervenção manual do proprietário para cada comando.

A filosofia é simples: **soltar as rédeas da Clara**, dando autonomia operacional sem abrir mão de recuperação, backups e rastreabilidade.

---

## 1. Preparação da VPS

Sistema recomendado:

- Debian 12
- VPS dedicada à Clara
- acesso SSH
- console de emergência do provedor
- snapshots disponíveis no provedor

Atualização inicial:

```bash
apt update && apt upgrade -y
apt install -y sudo git curl ca-certificates ufw
```

Antes de entregar autonomia à Clara, manter uma conta administrativa separada, pertencente exclusivamente ao proprietário, para recuperação.

---

## 2. Criar o usuário Clara

Criar um usuário próprio:

```bash
adduser clara
usermod -aG sudo clara
```

A Clara precisa instalar programas, alterar configurações, reiniciar serviços e resolver problemas sem esperar alguém entrar manualmente na VPS.

Para isso:

```bash
visudo
```

Adicionar:

```text
clara ALL=(ALL:ALL) NOPASSWD: ALL
```

Validar:

```bash
visudo -c
```

Na prática, a Clara terá autonomia administrativa completa por meio do sudo.

---

## 3. SSH e recuperação

Criar uma chave SSH exclusiva para a Clara.

Manter separadamente:

1. chave/credencial operacional da Clara;
2. credencial administrativa de emergência do proprietário;
3. console de recuperação do provedor da VPS.

Depois de confirmar que o acesso alternativo funciona:

- usar autenticação SSH por chave;
- desativar login remoto direto de root;
- manter a conta administrativa de emergência;
- manter snapshot externo antes de mudanças estruturais importantes.

A ideia não é limitar a Clara. É garantir que exista uma porta de recuperação caso uma configuração de rede, SSH ou firewall dê errado.

---

## 4. Instalar o Hermes Agent

Entrar como Clara:

```bash
su - clara
```

Instalar o Hermes Agent:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
source ~/.bashrc
```

Validar:

```bash
hermes
hermes doctor
```

Configurar o modelo:

```bash
hermes model
```

As chaves de API devem ficar em arquivos de ambiente ou armazenamento seguro da VPS, nunca dentro de repositórios públicos.

Antes de conectar canais e automações, testar uma conversa normal com o Hermes.

---

## 5. Hermes Gateway

Depois do agente funcionando, ativar e validar o Hermes Gateway.

O Gateway será a camada que permitirá à Clara trabalhar continuamente e receber solicitações pelos canais configurados.

Arquitetura básica:

```text
Usuário
   ↓
Telegram / WhatsApp / E-mail / API
   ↓
Hermes Gateway
   ↓
Clara (Hermes Agent)
   ↓
Ferramentas / MCPs / Terminal / sudo
   ↓
Debian 12
   ↓
Nginx / Docker / aplicações / serviços
```

---

## 6. Autonomia administrativa

A Clara poderá, quando uma tarefa exigir:

- instalar pacotes;
- criar ambientes Python;
- instalar Node.js e dependências;
- configurar Docker;
- criar containers;
- configurar Nginx;
- emitir ou renovar certificados;
- criar serviços;
- reiniciar serviços;
- consultar logs;
- editar arquivos;
- criar diretórios;
- clonar repositórios;
- executar Git;
- instalar novas ferramentas;
- atualizar componentes;
- diagnosticar erros;
- corrigir configurações.

O objetivo é evitar o fluxo:

```text
Clara sugere comando
→ proprietário entra por SSH
→ proprietário copia comando
→ proprietário executa
→ volta para Clara
```

O fluxo desejado é:

```text
Proprietário pede uma tarefa
→ Clara analisa
→ Clara instala o que precisar
→ Clara configura
→ Clara testa
→ Clara corrige se necessário
→ Clara informa o resultado
```

---

## 7. E-mail próprio da Clara

Criar uma identidade de e-mail exclusiva para a Clara.

Ela deverá poder:

- receber mensagens;
- enviar mensagens;
- responder contatos;
- receber confirmações de serviços;
- receber alertas;
- trabalhar com códigos e notificações quando apropriado;
- comunicar resultados e solicitações.

A integração pode ser feita por IMAP/SMTP ou por uma ferramenta compatível, como Himalaya, dependendo do provedor escolhido.

Credenciais devem permanecer fora do GitHub.

---

## 8. Domínio próprio

Destinar um domínio para a Clara.

O domínio continua pertencendo ao proprietário, mas a Clara terá autonomia para administrar a infraestrutura associada a ele.

Exemplos futuros:

```text
www.dominio.com
blog.dominio.com
api.dominio.com
app.dominio.com
status.dominio.com
files.dominio.com
```

Ela poderá criar novos subdomínios conforme instalar novos serviços.

---

## 9. Cloudflare

Adicionar o domínio à Cloudflare.

Criar um **API Token exclusivo para a Clara**, limitado somente à zona do domínio dela.

A Clara poderá:

- consultar DNS;
- criar registros;
- editar registros;
- remover registros;
- apontar subdomínios;
- configurar serviços relacionados ao domínio quando autorizado pelo token.

Não entregar uma Global API Key com acesso a todos os domínios.

O princípio é:

```text
Clara tem liberdade total dentro do domínio dela.
Outros domínios permanecem isolados.
```

---

## 10. GitHub

Usar uma conta GitHub destinada à Clara.

Configurar autenticação segura na VPS, preferencialmente com chave SSH própria.

A Clara poderá:

- criar e administrar repositórios;
- clonar projetos;
- criar arquivos;
- editar código;
- fazer commits;
- fazer push;
- manter documentação;
- publicar conteúdo;
- versionar configurações que não contenham segredos;
- trabalhar nos projetos dela.

Segredos, tokens, senhas e chaves privadas nunca devem ser commitados.

---

## 11. GitHub Pages e site da Clara

Criar um repositório para o site/blog da Clara.

Fluxo:

```text
Clara escreve
↓
salva no repositório
↓
commit + push
↓
GitHub Pages publica
↓
domínio personalizado exibe o conteúdo
```

A própria Clara poderá administrar o conteúdo e, com o token Cloudflare dela, ajustar o DNS necessário para o domínio personalizado.

---

## 12. Canais de comunicação

### Telegram

Configurar um bot dedicado para conversar diretamente com a Clara.

Ele pode funcionar como canal administrativo e de comando.

### WhatsApp

Pode ser integrado posteriormente para atendimento, automações ou operação comercial.

### E-mail

Além de comunicação externa, o e-mail pode funcionar como outro canal de trabalho da Clara.

---

## 13. MCPs, APIs e novas ferramentas

A instalação deve nascer preparada para crescimento.

Quando surgir uma necessidade nova, a Clara poderá:

1. identificar a ferramenta necessária;
2. consultar documentação;
3. instalar dependências;
4. configurar a integração;
5. armazenar credenciais de forma segura;
6. testar;
7. documentar;
8. colocar o serviço em funcionamento.

Exemplos:

- Gmail;
- Google Calendar;
- Google Drive;
- GitHub;
- Cloudflare;
- Listmonk;
- n8n;
- Omie;
- APIs próprias;
- bancos de dados;
- serviços de voz;
- ferramentas de mídia;
- MCPs.

---

## 14. Credenciais e segredos

Criar uma estrutura organizada para segredos.

Nunca colocar em repositório público:

- API keys;
- tokens;
- senhas;
- cookies;
- chaves SSH privadas;
- credenciais de banco;
- credenciais SMTP/IMAP;
- tokens Cloudflare.

Usar permissões restritas nos arquivos que armazenarem esses dados.

Exemplo:

```bash
chmod 600 arquivo-de-segredos
```

---

## 15. Logs e auditoria

Manter registro das atividades administrativas relevantes.

Isso ajuda a entender:

- o que foi instalado;
- o que foi alterado;
- quando ocorreu;
- por que ocorreu;
- qual serviço apresentou erro;
- como a Clara corrigiu o problema.

A auditoria não deve impedir a autonomia. Ela serve para observabilidade e recuperação.

---

## 16. Backups e snapshots

Manter pelo menos:

- snapshot periódico da VPS;
- backup das configurações importantes;
- backup dos dados persistentes;
- repositórios Git para código e documentação;
- cópia externa do que não puder ser reconstruído.

Antes de mudanças estruturais de alto impacto, criar snapshot quando possível.

---

## 17. Conta de emergência

A Clara não deve ser a única forma de entrar na VPS.

Manter uma conta administrativa separada para o proprietário.

Essa conta não é para o trabalho diário.

Ela existe para situações como:

- SSH quebrado;
- firewall configurado incorretamente;
- Hermes indisponível;
- serviço essencial corrompido;
- credencial da Clara perdida;
- necessidade de recuperação manual.

---

## 18. Ordem recomendada da implantação

1. Criar a VPS Debian 12.
2. Atualizar o sistema.
3. Criar a conta administrativa de emergência.
4. Criar o usuário `clara`.
5. Configurar SSH por chave.
6. Liberar sudo administrativo para Clara.
7. Configurar firewall básico.
8. Instalar Hermes Agent.
9. Configurar modelo e API.
10. Testar Hermes.
11. Configurar Hermes Gateway.
12. Configurar Telegram.
13. Criar e integrar o e-mail da Clara.
14. Registrar/destinar o domínio.
15. Adicionar domínio à Cloudflare.
16. Criar token Cloudflare exclusivo para a zona.
17. Configurar GitHub da Clara.
18. Configurar GitHub Pages.
19. Apontar o domínio personalizado.
20. Instalar MCPs e integrações iniciais.
21. Configurar logs.
22. Configurar snapshots e backups.
23. Testar recuperação.
24. Documentar toda a instalação.

---

## 19. Arquitetura final

```text
                    ┌─────────────────────┐
                    │      Proprietário   │
                    └──────────┬──────────┘
                               │
             ┌─────────────────┼─────────────────┐
             │                 │                 │
         Telegram           E-mail          Outros canais
             │                 │                 │
             └─────────────────┼─────────────────┘
                               ↓
                    ┌─────────────────────┐
                    │   Hermes Gateway    │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │        Clara        │
                    │    Hermes Agent     │
                    └──────────┬──────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
       Terminal              MCPs                APIs
       + sudo                  │                    │
          │                    │                    │
          ↓                    ↓                    ↓
       Debian 12        Google / GitHub      Cloudflare /
          │             serviços etc.        outros serviços
          ↓
   Nginx / Docker /
   aplicações / serviços

          Clara também mantém:
          • e-mail próprio
          • domínio próprio
          • DNS/subdomínios
          • GitHub
          • GitHub Pages
          • documentação
          • backups operacionais
```

---

## 20. Princípio operacional

A nova Clara não será apenas uma interface que entrega comandos para alguém executar.

Ela será uma agente com ambiente próprio e capacidade de agir.

Quando precisar de uma ferramenta para concluir uma tarefa, poderá instalar, configurar, testar e usar essa ferramenta.

Quando precisar publicar algo, poderá trabalhar no GitHub.

Quando precisar colocar um serviço na internet, poderá configurar o servidor e administrar o DNS do domínio destinado a ela.

Quando precisar se comunicar, terá seus próprios canais.

O proprietário continua com a autoridade final sobre a infraestrutura, o domínio e as contas, além de manter acesso de emergência e backups.

A meta é uma Clara **autônoma, organizada, recuperável e capaz de trabalhar de verdade**.
