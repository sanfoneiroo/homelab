# LAB 7 - Ambiente de Desenvolvimento Remoto com VPN e Code Server

## Objetivo

Criar um ambiente de desenvolvimento remoto acessível de qualquer lugar através de VPN, permitindo programar, editar arquivos e administrar o homelab utilizando apenas um navegador web.

O cenário foi pensado para viagens ou acesso remoto eventual. Com um roteador portátil Cudy TR1200 conectado à VPN, é possível acessar a infraestrutura doméstica de forma segura mesmo atrás de CGNAT, Wi-Fi público ou duplo NAT.

---

## Pré-requisitos

Este laboratório utiliza componentes configurados anteriormente:

* VPN ZeroTier
* VM Debian
* CasaOS

Com a VPN funcionando, todos os serviços da VM tornam-se acessíveis através da rede virtual privada.

---

# Instalação do Code Server

Para disponibilizar um ambiente completo de desenvolvimento acessível pelo navegador, foi utilizado o aplicativo Big Bear Code Server disponível no CasaOS.

Instalar pelo App Store do CasaOS:

Apps → BigBearCasaOS → Code Server

Após a instalação, o diretório de trabalho padrão fica armazenado dentro da estrutura do aplicativo.

Foi criado um link simbólico para facilitar o acesso:

```bash
ln -s /home/vm10/DATA/AppData/big-bear-code-server/config/workspace /home/vm10/workspace
```

Agora o workspace pode ser acessado diretamente pelo usuário.

Verificação:

```bash
ls -l ~/workspace
```

---

# Instalação do GitHub CLI

O GitHub CLI permite autenticação, clonagem de repositórios e diversas operações sem utilizar o navegador.

Adicionar o repositório oficial:

```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
| sudo gpg --dearmor -o /usr/share/keyrings/githubcli-archive-keyring.gpg
```

Adicionar a fonte do pacote:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
| sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
```

Atualizar repositórios:

```bash
sudo apt update
```

Instalar:

```bash
sudo apt install gh
```

Verificar instalação:

```bash
gh --version
```

Autenticar:

```bash
gh auth login
```

Seguir o procedimento exibido pelo terminal.

---

# Clonando Repositórios

Após autenticação:

```bash
cd ~/workspace
```

```bash
git clone https://github.com/usuario/repositorio.git
```

Todos os projetos ficam disponíveis imediatamente no Code Server.

---

# Organização dos Projetos

Estrutura utilizada:

```text
workspace/
├── homelab/
├── projeto1/
├── projeto2/
└── outros-projetos/
```

Isso facilita a navegação tanto pelo terminal quanto pela interface web do VS Code.

---

# Atalhos para Workspaces

Para acelerar a navegação via SSH, podem ser criados aliases no shell.

Editar:

```bash
nano ~/.bashrc
```

Adicionar:

```bash
alias homelab='cd ~/workspace/homelab'
alias projeto1='cd ~/workspace/projeto1'
alias projeto2='cd ~/workspace/projeto1'
```

Aplicar:

```bash
source ~/.bashrc
```

Agora basta executar:

```bash
projeto1
```

---

# Publicação do Code Server via Proxy Reverso

Após a instalação do Code Server, o acesso inicial normalmente ocorre através de uma porta específica, por exemplo:

```text
http://IP_DA_VM:8080
```

Para integrar o serviço ao ambiente do homelab, foi utilizado o Nginx como proxy reverso.

Criar o arquivo:

```bash
sudo nano /etc/nginx/sites-available/code-server
```

Conteúdo:

```nginx
server {
    listen 80;

    server_name code-server.lab;

    location / {
        proxy_pass http://127.0.0.1:8080;

        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_read_timeout 86400;
        proxy_send_timeout 86400;
    }
}
```

Criar o link simbólico:

```bash
sudo ln -s /etc/nginx/sites-available/code-server \
/etc/nginx/sites-enabled/
```

Verificar a configuração:

```bash
sudo nginx -t
```

Recarregar o serviço:

```bash
sudo systemctl reload nginx
```

## Importância dos WebSockets

O Code Server utiliza WebSockets para comunicação entre navegador e servidor.

Sem as diretivas:

```nginx
proxy_http_version 1.1;

proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

o navegador pode exibir erros como:

```text
The workbench failed to connect to the server
WebSocket close with status code 1006
```

A inclusão desses parâmetros permite que o VS Code Web funcione corretamente através do proxy reverso.

---

# Configuração do DNS Local

Adicionar ao arquivo:

```bash
sudo nano /etc/dnsmasq.conf
```

Entrada:

```text
address=/code-server.lab/192.168.10.100
```

Substituindo o IP pelo endereço da VM.

Reiniciar o DNS:

```bash
sudo systemctl restart dnsmasq
```

Agora o serviço pode ser acessado diretamente através de:

```text
http://code-server.lab
```

sem necessidade de informar portas manualmente.

---

# Cenário de Uso

Exemplo durante uma viagem:

1. O computador principal permanece ligado em casa.
2. A VM Debian permanece conectada à VPN ZeroTier.
3. O roteador portátil TR1200 conecta-se à internet do local.
4. O TR1200 ingressa automaticamente na VPN.
5. Notebook, celular ou tablet conectam-se ao TR1200.
6. O usuário acessa:

   * CasaOS
   * Code Server
   * Dashboard do Homelab
   * Demais serviços internos

Tudo sem abertura de portas ou exposição direta à internet.

---

# Vantagens da Solução

* Sem port forwarding.
* Funciona atrás de CGNAT.
* Funciona em Wi-Fi público.
* Ambiente de desenvolvimento acessível pelo navegador.
* Repositórios sincronizados com GitHub.
* Administração remota do homelab.
* Baixo custo operacional.
* Maior segurança por utilizar VPN privada.

---

# Resultado

O Lab 7 transforma o homelab em uma estação de desenvolvimento remota. Com apenas um navegador e acesso à VPN, é possível editar código, acessar repositórios GitHub, administrar serviços e continuar projetos de qualquer lugar, mantendo todos os recursos centralizados na infraestrutura doméstica.