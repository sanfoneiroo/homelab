# LAB 9 - Servidor Git Privado com Gitea

## Objetivo

Implantar um servidor Git privado utilizando Gitea para centralizar o armazenamento de projetos, documentação e configurações do homelab.

Ao final deste laboratório, os repositórios passam a ser acessados através de um endereço amigável:

```text
http://gitea.lab
```

sem necessidade de informar portas ou endereços IP.

---

# Arquitetura

Antes:

```text
Code-Server
     │
     ▼
Arquivos locais
```

Depois:

```text
Code-Server
     │
     ▼
Gitea
     │
     ▼
Repositórios Git
```

Com acesso via DNS interno:

```text
Cliente LAN
     │
     ▼
DNS Interno
     │
gitea.lab → 192.168.10.100
     │
     ▼
Nginx
     │
     ▼
localhost:3000
     │
     ▼
Gitea
```

---

# Instalação do Gitea

O Gitea foi instalado através da loja de aplicações do CasaOS.

Após a instalação, o serviço ficou disponível em:

```text
http://192.168.10.100:3000
```

Na primeira execução foi criado o usuário administrador e definida a senha de acesso.

---

# Criação do Primeiro Repositório

Após realizar login:

1. Clicar em **+**
2. Selecionar **New Repository**
3. Definir o nome do repositório
4. Marcar:

   * Initialize Repository
   * Add README.md

Exemplo:

```text
lab8-gitea
```

---

# Instalação do Git

Verificar se o Git está instalado:

```bash
git --version
```

Caso necessário:

```bash
sudo apt update
sudo apt install git -y
```

Configurar identidade:

```bash
git config --global user.name "Seu Nome"
git config --global user.email "usuario@lab.local"
```

---

# Integração com Code-Server

Criar diretório para os projetos:

```bash
mkdir -p ~/repos
cd ~/repos
```

Clonar o repositório:

```bash
git clone http://gitea.lab/usuario/lab8-gitea.git
```

Entrar na pasta:

```bash
cd lab8-gitea
```

Abrir a pasta dentro do Code-Server.

---

# Fluxo Básico de Trabalho

Adicionar arquivos:

```bash
git add .
```

Criar commit:

```bash
git commit -m "Primeiro commit"
```

Enviar alterações:

```bash
git push
```

Atualizando a página do Gitea será possível visualizar imediatamente os arquivos enviados.

---

# Token de Acesso

Em vez de utilizar a senha da conta, recomenda-se gerar um token pessoal.

Menu:

```text
Settings
└── Applications
    └── Generate Token
```

Ao realizar operações Git:

```text
Usuário: nome_da_conta
Senha: token_gerado
```

---

# Publicação com Nginx e DNS

Para eliminar a necessidade da porta 3000 foi criado um script de automação responsável por:

* Criar a configuração do proxy reverso.
* Habilitar o site no Nginx.
* Validar a configuração.
* Recarregar o serviço.
* Criar o registro DNS automaticamente.
* Reiniciar o dnsmasq.

---

# Script de Automação

```bash
#!/bin/bash

echo "=== Configurando proxy reverso para Gitea ==="

# Cria configuração do site
sudo tee /etc/nginx/sites-available/gitea > /dev/null <<'EOF'
server {
    listen 80;

    server_name gitea.lab;

    location / {

        proxy_pass http://127.0.0.1:3000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
EOF

# Habilita site
sudo ln -sf /etc/nginx/sites-available/gitea \
            /etc/nginx/sites-enabled/gitea

# Testa configuração
sudo nginx -t

if [ $? -eq 0 ]; then
    sudo systemctl reload nginx
    echo "Proxy reverso configurado com sucesso."
else
    echo "Erro na configuração do Nginx."
fi

# Configura DNS
if ! grep -q "gitea.lab" /etc/dnsmasq.conf; then
    echo "address=/gitea.lab/192.168.10.100" | sudo tee -a /etc/dnsmasq.conf
fi

sudo systemctl restart dnsmasq
```

---

# Funcionamento do Script

O script cria automaticamente o arquivo:

```text
/etc/nginx/sites-available/gitea
```

com a configuração:

```nginx
server {
    listen 80;

    server_name gitea.lab;

    location / {
        proxy_pass http://127.0.0.1:3000;
    }
}
```

Em seguida:

1. Habilita o site.
2. Valida a configuração do Nginx.
3. Recarrega o serviço.

Após a configuração do proxy, o script verifica se o domínio já existe no arquivo:

```text
/etc/dnsmasq.conf
```

Caso não exista, adiciona:

```conf
address=/gitea.lab/192.168.10.100
```

Por fim, reinicia o dnsmasq para aplicar a nova configuração.

A verificação evita registros duplicados e permite executar o script mais de uma vez sem gerar entradas repetidas.

---

# Ajuste do ROOT_URL

Para que o Gitea gere URLs corretas, ajustar o arquivo de configuração:

```ini
ROOT_URL = http://gitea.lab/
```

Após a alteração, reiniciar o container do Gitea.

---

# Validação

Testar resolução DNS:

```bash
ping gitea.lab
```

Resultado esperado:

```text
PING gitea.lab (192.168.10.100)
```

Testar acesso HTTP:

```text
http://gitea.lab
```

Testar clonagem:

```bash
git clone http://gitea.lab/usuario/repositorio.git
```

---

# Sincronização com GitHub (Opcional)

O mesmo repositório pode possuir múltiplos remotos.

Exemplo:

```bash
git remote add github https://github.com/usuario/repositorio.git
```

Enviar para o Gitea:

```bash
git push origin main
```

Enviar para o GitHub:

```bash
git push github main
```

Dessa forma o Gitea permanece como repositório principal enquanto o GitHub atua como espelho ou backup externo.

---

# Resultado

O homelab passa a oferecer:

* Servidor Git privado.
* Interface web semelhante ao GitHub.
* Integração com Code-Server.
* Controle de versão para projetos e documentação.
* Proxy reverso com Nginx.
* Resolução DNS local.
* Possibilidade de sincronização com GitHub.

Serviços disponíveis:

```text
dashboard.lab
casaos.lab
netdata.lab
gitea.lab
```

Com este laboratório, toda a documentação e configuração do homelab passa a possuir histórico de versões, facilitando backup, organização e recuperação de alterações.
