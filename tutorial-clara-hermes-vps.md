# Tutorial completo: instalar a Clara com Hermes Agent em uma VPS Debian 12

> Objetivo: sair de uma VPS Debian 12 recém-criada e chegar a uma **Clara funcional no Hermes Agent**, com usuário próprio, acesso SSH, `sudo` administrativo sem senha e um conjunto inicial de ferramentas. Depois desse ponto, a própria Clara poderá instalar e configurar o restante conforme as tarefas surgirem.

## Antes de começar

Este tutorial assume:

- VPS nova com Debian 12;
- acesso inicial como `root`;
- um computador seu com cliente SSH;
- uma chave pública SSH sua para autorizar o login como `clara`;
- uma chave/API ou método de autenticação para o provedor de IA que será usado pelo Hermes.

Guarde também o acesso ao console web da empresa da VPS. Ele será a porta de emergência caso alguma configuração de SSH ou rede dê errado.

---

# Etapa 1: primeira entrada na VPS

No seu computador, entre na VPS pela primeira vez:

```bash
ssh root@IP_DA_VPS
```

Exemplo:

```bash
ssh root@203.0.113.10
```

Confirme o sistema:

```bash
cat /etc/os-release
```

Confira usuário e hostname:

```bash
whoami
hostnamectl
```

O `whoami` deve retornar:

```text
root
```

---

# Etapa 2: atualizar o Debian

Atualize a lista de pacotes:

```bash
apt update
```

Atualize os pacotes instalados:

```bash
apt full-upgrade -y
```

Remova dependências que não são mais necessárias:

```bash
apt autoremove -y
apt autoclean
```

Se uma atualização importante solicitar reinicialização:

```bash
reboot
```

Aguarde a VPS voltar e conecte novamente:

```bash
ssh root@IP_DA_VPS
```

---

# Etapa 3: instalar as ferramentas básicas do servidor

Instale o conjunto inicial:

```bash
apt install -y \
  sudo \
  git \
  curl \
  wget \
  xz-utils \
  ca-certificates \
  gnupg \
  unzip \
  zip \
  jq \
  nano \
  vim \
  tmux \
  htop \
  tree \
  rsync \
  openssh-client \
  openssh-server \
  ufw \
  fail2ban \
  sqlite3 \
  build-essential
```

Verifique algumas ferramentas:

```bash
git --version
curl --version
jq --version
tmux -V
sqlite3 --version
```

O instalador oficial do Hermes cuida das dependências próprias dele, incluindo Python, Node.js, ripgrep e FFmpeg quando necessário. Não precisamos criar manualmente uma instalação paralela dessas dependências.

---

# Etapa 4: configurar fuso horário

Para a Clara operar no horário de Castanhal/Belém:

```bash
timedatectl set-timezone America/Belem
```

Confira:

```bash
timedatectl
```

---

# Etapa 5: criar o usuário Clara

Crie o usuário:

```bash
adduser clara
```

O Debian solicitará uma senha. Use uma senha forte e guarde-a em local seguro.

Adicione Clara ao grupo `sudo`:

```bash
usermod -aG sudo clara
```

Confira:

```bash
id clara
```

A saída deverá mostrar o grupo `sudo`.

---

# Etapa 6: liberar sudo administrativo sem senha

Crie uma regra exclusiva:

```bash
visudo -f /etc/sudoers.d/clara
```

Adicione exatamente:

```text
clara ALL=(ALL:ALL) NOPASSWD: ALL
```

Salve e saia.

Proteja o arquivo:

```bash
chmod 440 /etc/sudoers.d/clara
```

Valide a configuração antes de continuar:

```bash
visudo -c
```

O resultado precisa indicar que os arquivos foram analisados corretamente.

Teste:

```bash
su - clara
sudo whoami
```

O resultado deve ser:

```text
root
```

Volte para root:

```bash
exit
```

Isso dá à Clara a autonomia administrativa que queremos. Na prática, ela poderá instalar pacotes, alterar serviços e administrar a VPS usando `sudo`.

---

# Etapa 7: preparar o SSH da Clara

Crie o diretório SSH:

```bash
install -d -m 700 -o clara -g clara /home/clara/.ssh
```

Agora abra o arquivo de chaves autorizadas:

```bash
nano /home/clara/.ssh/authorized_keys
```

Cole **a chave pública SSH do seu computador**, por exemplo uma linha iniciada por:

```text
ssh-ed25519 AAAA...
```

Não coloque uma chave privada nesse arquivo.

Depois:

```bash
chown clara:clara /home/clara/.ssh/authorized_keys
chmod 600 /home/clara/.ssh/authorized_keys
```

Em outro terminal do seu computador, sem fechar a sessão root atual, teste:

```bash
ssh clara@IP_DA_VPS
```

Depois:

```bash
whoami
sudo whoami
```

Você deverá receber:

```text
clara
root
```

**Só continue depois que esse login estiver funcionando.**

---

# Etapa 8: proteger o SSH

Antes de alterar o SSH, mantenha aberta a sessão atual e confirme que o console de emergência do provedor funciona.

Crie um arquivo específico:

```bash
sudo nano /etc/ssh/sshd_config.d/99-clara-security.conf
```

Adicione:

```text
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

Valide antes de recarregar:

```bash
sudo sshd -t
```

Se o comando não mostrar erro:

```bash
sudo systemctl reload ssh
```

Abra **mais um terminal** e teste novamente:

```bash
ssh clara@IP_DA_VPS
```

Não encerre a sessão antiga antes de confirmar o novo acesso.

---

# Etapa 9: firewall básico

Libere SSH primeiro:

```bash
sudo ufw allow OpenSSH
```

Como a VPS provavelmente hospedará aplicações web:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Ative:

```bash
sudo ufw enable
```

Confira:

```bash
sudo ufw status verbose
```

Não abra portas adicionais sem necessidade. Quando um serviço novo precisar de uma porta pública, a própria Clara poderá avaliar e configurar.

---

# Etapa 10: ativar Fail2ban

Ative o serviço:

```bash
sudo systemctl enable --now fail2ban
```

Confira:

```bash
sudo systemctl status fail2ban --no-pager
```

---

# Etapa 11: entrar definitivamente como Clara

A partir daqui, trabalhe como:

```bash
ssh clara@IP_DA_VPS
```

Confirme:

```bash
whoami
pwd
sudo whoami
```

Esperado:

```text
clara
/home/clara
root
```

---

# Etapa 12: instalar o Hermes Agent pela fonte oficial

O comando oficial para Linux é:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

**Execute como usuário `clara`, sem colocar `sudo` antes do instalador.**

O instalador oficial cria uma instalação por usuário e cuida das dependências do Hermes.

Quando terminar:

```bash
source ~/.bashrc
```

Garanta que o diretório local de binários esteja no PATH:

```bash
grep -q 'HOME/.local/bin' ~/.bashrc || echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Confira:

```bash
which hermes
hermes --version
```

A instalação por usuário normalmente mantém os arquivos do Hermes dentro de:

```text
/home/clara/.hermes/
```

e o launcher em:

```text
/home/clara/.local/bin/hermes
```

---

# Etapa 13: diagnóstico do Hermes

Execute:

```bash
hermes doctor
```

Corrija qualquer problema indicado antes de avançar.

Também podemos conferir:

```bash
ls -la ~/.hermes
```

---

# Etapa 14: configurar o modelo de IA

Execute:

```bash
hermes model
```

Escolha o provedor e o modelo que serão usados pela Clara.

Outra opção é executar o assistente completo:

```bash
hermes setup
```

Se for usar Nous Portal, o Hermes também oferece:

```bash
hermes setup --portal
```

Em uma VPS acessada remotamente, autenticações OAuth podem exigir encaminhamento de porta ou fluxo de autenticação apropriado para servidor remoto.

Não coloque chaves de API em arquivos públicos ou no GitHub.

---

# Etapa 15: primeiro teste da Clara

Antes de instalar gateway, integrações, WhatsApp, Cloudflare ou qualquer outra coisa, confirme que o agente consegue conversar normalmente.

Execute:

```bash
hermes
```

Ou faça um teste direto:

```bash
hermes chat -q "Responda apenas: Clara está funcionando."
```

O objetivo desta etapa é provar quatro coisas:

1. Hermes está instalado;
2. o modelo está configurado;
3. a autenticação funciona;
4. Clara consegue receber uma tarefa e responder.

Se isso não funcionar, pare aqui e corrija antes de adicionar novas camadas.

---

# Etapa 16: verificar as ferramentas do Hermes

Execute:

```bash
hermes tools
```

O Hermes possui ferramentas próprias e pode ser configurado conforme a operação desejada.

A Clara já terá acesso ao terminal do ambiente e, pelo sistema operacional, o usuário `clara` possui `sudo` administrativo.

Faça um teste controlado dentro do Hermes:

```text
Verifique o sistema operacional, espaço em disco, memória disponível e versão do Git. Não altere nenhuma configuração. Apenas me entregue o diagnóstico.
```

Depois, um teste administrativo simples:

```text
Verifique se o pacote tree está instalado. Se já estiver, apenas confirme. Se não estiver, instale usando sudo e depois confirme a versão.
```

Se a Clara conseguir diagnosticar e executar a tarefa, a autonomia básica está funcionando.

---

# Etapa 17: instalar Nginx e Certbot

Esses dois componentes serão úteis quando Clara começar a publicar APIs, painéis e serviços:

```bash
sudo apt install -y nginx certbot python3-certbot-nginx
```

Ative o Nginx:

```bash
sudo systemctl enable --now nginx
```

Confira:

```bash
sudo systemctl status nginx --no-pager
```

Não emita certificado ainda. Isso será feito quando o domínio e o DNS estiverem apontados corretamente.

---

# Etapa 18: instalar Docker

Docker será útil para a Clara subir bancos, aplicações e serviços isolados.

Use o repositório oficial do Docker:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Adicione o repositório:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Atualize:

```bash
sudo apt update
```

Instale:

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Ative:

```bash
sudo systemctl enable --now docker
```

Teste com sudo:

```bash
sudo docker run --rm hello-world
```

Como Clara já possui `sudo`, não é necessário colocá-la no grupo `docker`. Isso evita criar mais um caminho de privilégio administrativo fora do sudo.

---

# Etapa 19: ferramentas que já estarão disponíveis

Ao final, a VPS terá uma base prática para a Clara trabalhar:

```text
Sistema
├── Debian 12 atualizado
├── SSH
├── sudo
├── UFW
├── Fail2ban
└── ferramentas de diagnóstico

Desenvolvimento
├── Git
├── curl / wget
├── jq
├── build-essential
├── SQLite
├── zip / unzip
├── rsync
└── ferramentas instaladas pelo próprio Hermes

Operação
├── tmux
├── htop
├── tree
├── nano
└── vim

Web
├── Nginx
└── Certbot

Containers
├── Docker Engine
├── Docker Buildx
└── Docker Compose

IA
└── Hermes Agent
    ├── modelo configurado
    ├── terminal
    ├── ferramentas
    └── acesso ao sistema como clara + sudo
```

---

# Etapa 20: habilitar persistência para serviços do usuário Clara

Como o Hermes Gateway pode funcionar como serviço de usuário e continuar ativo mesmo sem uma sessão SSH aberta, habilite lingering:

```bash
sudo loginctl enable-linger clara
```

Confira:

```bash
loginctl show-user clara -p Linger
```

Esperado:

```text
Linger=yes
```

---

# Etapa 21: configurar o Hermes Gateway

Agora que a Clara básica já funciona:

```bash
hermes gateway setup
```

Siga o assistente para configurar os canais desejados.

Depois consulte os comandos disponíveis da versão instalada:

```bash
hermes gateway --help
```

Não copie comandos antigos de outras versões sem conferir o `--help` da versão instalada.

---

# Etapa 22: snapshot antes de entregar a autonomia

Neste ponto, faça um snapshot pelo painel do provedor da VPS.

Nome sugerido:

```text
clara-base-hermes-funcional
```

Esse snapshot representa o ponto seguro:

- Debian funcionando;
- SSH funcionando;
- usuário Clara funcionando;
- sudo funcionando;
- firewall funcionando;
- Hermes instalado;
- modelo funcionando;
- ferramentas básicas instaladas;
- Docker instalado;
- Nginx instalado.

Se uma experiência futura quebrar o servidor, existe um ponto limpo para recuperação.

---

# Etapa 23: teste final

Entre novamente:

```bash
ssh clara@IP_DA_VPS
```

Execute:

```bash
hermes doctor
sudo whoami
git --version
docker --version
sudo systemctl is-active nginx
sudo systemctl is-active docker
sudo ufw status
```

Depois:

```bash
hermes
```

Peça:

```text
Faça um diagnóstico desta VPS. Verifique CPU, memória, disco, sistema operacional, serviços principais e ferramentas de desenvolvimento disponíveis. Não faça alterações. Depois me diga se o ambiente está pronto para você trabalhar.
```

Se o diagnóstico estiver correto, a base está pronta.

---

# Daqui para frente, Clara assume

Este tutorial termina de propósito neste ponto.

Não precisamos instalar antecipadamente dezenas de ferramentas que talvez nunca sejam usadas.

A partir daqui, quando surgir uma necessidade, podemos pedir diretamente à Clara.

Exemplos:

```text
Clara, preciso publicar uma API Python neste servidor. Escolha uma estrutura adequada, instale o que estiver faltando, configure como serviço, coloque atrás do Nginx e teste.
```

```text
Clara, configure meu domínio para este serviço. Antes de alterar qualquer coisa, verifique o estado atual e faça backup das configurações que serão modificadas.
```

```text
Clara, precisamos integrar o GitHub. Verifique o que falta, configure a autenticação e teste acesso ao repositório.
```

```text
Clara, precisamos de uma ferramenta para converter PDFs em imagens. Escolha uma opção adequada para Debian, instale e faça um teste.
```

Esse é exatamente o ponto da arquitetura: **não preparar manualmente todas as possibilidades, mas preparar a Clara para preparar o próprio ambiente.**

---

# Checklist final

- [ ] Debian 12 atualizado
- [ ] usuário `clara` criado
- [ ] SSH por chave funcionando
- [ ] login remoto de root desativado
- [ ] senha SSH desativada
- [ ] `sudo` sem senha funcionando para Clara
- [ ] UFW ativo
- [ ] Fail2ban ativo
- [ ] Hermes Agent instalado pelo instalador oficial
- [ ] `hermes doctor` funcionando
- [ ] modelo de IA configurado
- [ ] primeira conversa funcionando
- [ ] ferramentas do Hermes verificadas
- [ ] Nginx instalado
- [ ] Certbot instalado
- [ ] Docker instalado
- [ ] Docker Compose instalado
- [ ] lingering habilitado para Clara
- [ ] Hermes Gateway configurado
- [ ] snapshot da base funcional criado
- [ ] console de emergência do provedor testado

## Referências oficiais

- Hermes Agent: https://hermes-agent.nousresearch.com/
- Código oficial Hermes Agent: https://github.com/NousResearch/hermes-agent
- Docker Engine para Debian: https://docs.docker.com/engine/install/debian/

---

**Projeto:** Clara autônoma  
**Base:** Hermes Agent + Debian 12  
**Objetivo:** entregar à Clara uma VPS funcional e administrável para que, a partir desse ponto, ela mesma possa instalar, configurar e manter as ferramentas necessárias.
