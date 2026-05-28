# CONFIGURAÇÃO DE SERVIÇOS NO BOOT COM SYSTEMD

Neste laboratório foi utilizada a infraestrutura de serviços do Linux através do `systemd` para automatizar a inicialização de aplicações e serviços durante o boot da VM Debian.

A proposta foi transformar scripts e comandos utilizados manualmente nos laboratórios anteriores em serviços persistentes, permitindo que toda a infraestrutura do homelab seja iniciada automaticamente ao ligar a máquina virtual.

O primeiro caso foi a automação da conexão com a rede overlay utilizando ZeroTier. Para isso, foi criado um pequeno script responsável apenas por executar o comando de entrada na rede:

```bash
#!/bin/bash

zerotier-cli join ID_DA_REDE
```

Após tornar o script executável:

```bash
chmod +x zerotier.sh
```

foi criado um serviço do `systemd` apontando para esse script.

```bash
sudo nano /etc/systemd/system/zerotier.service
```

```ini
[Unit]
Description=Inicializacao do zerotier
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/home/vm10/homelab/scripts/zerotier.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Depois disso, os serviços foram recarregados e habilitados no boot:

```bash
sudo systemctl daemon-reload
sudo systemctl enable zerotier.service
sudo systemctl start zerotier.service
```

Os logs podem ser acompanhados com:

```bash
sudo journalctl -u zerotier.service -f
```

---

Também foi criado um serviço para inicializar automaticamente um servidor web simples utilizando o módulo `http.server` do Python.

Neste caso não foi necessário utilizar script intermediário, já que o próprio comando pode ser executado diretamente pelo `ExecStart`.

```bash
sudo nano /etc/systemd/system/webserver.service
```

```ini
[Unit]
Description=Servidor Web Python
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 -m http.server 80 --bind 0.0.0.0 --directory /home/vm10/homelab/web
WorkingDirectory=/home/vm10/homelab/web
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Ativação:

```bash
sudo systemctl daemon-reload
sudo systemctl enable webserver.service
sudo systemctl start webserver.service
```

Monitoramento:

```bash
sudo journalctl -u webserver.service -f
```

O parâmetro:

```bash
--bind 0.0.0.0
```

permite que o servidor aceite conexões externas pela LAN e também pela VPN overlay.

Já o parâmetro:

```bash
--directory
```

permite separar o diretório dos scripts do diretório efetivamente servido pelo webserver.

---

Com isso, ao iniciar a VM, toda a infraestrutura construída nos laboratórios anteriores passa a subir automaticamente.

Atualmente a inicialização automática inclui:

* VPN overlay com ZeroTier
* servidor web Python
* CasaOS
* servidor DNS local
* serviços auxiliares da VM Debian

Arquitetura resultante:

```text
VM Debian
│
├── zerotier.service
├── webserver.service
├── casaos.service
├── dns.service
│
├── VPN Overlay
├── Servidor HTTP Python
├── Serviços LAN
└── Inicialização Automática no Boot
```

O uso do `systemd` também facilita:

* reinício automático em caso de falha
* monitoramento centralizado via `journalctl`
* separação de responsabilidades entre serviços
* gerenciamento individual via `systemctl`
* organização mais próxima de ambientes reais de infraestrutura Linux

Com isso, a VM passa a funcionar como um pequeno servidor persistente dentro do homelab, iniciando toda a pilha de serviços automaticamente após o boot do sistema.
