
# Lab DNS Local + Debian VM + TR1200

## Objetivos
- criar um servidor DNS local para a LAN
- substituir acesso via IP por hostnames amigáveis
- manter fallback DNS caso a VM esteja offline
- estudar resolução de nomes em redes Linux
- implementar infraestrutura de DNS simples para homelab

O objetivo deste laboratório foi implementar um servidor DNS local dentro da rede LAN, permitindo acessar dispositivos e serviços utilizando nomes amigáveis ao invés de endereços IP.

Exemplo:

Antes:

```txt
http://192.168.10.2
```

Depois:

```txt
http://esp32.lab
```

Além da praticidade, isso aproxima a estrutura da rede de ambientes reais de infraestrutura e homelab.

---

# Provisionamento Debian

## Instalação do dnsmasq

Atualização do sistema:

```bash
sudo apt update
```

Instalação do serviço DNS:

```bash
sudo apt install dnsmasq
```

---

# Configuração do dnsmasq

Arquivo de configuração:

```bash
sudo micro /etc/dnsmasq.conf
```

Configuração utilizada:

```conf
no-resolv

server=8.8.8.8
server=1.1.1.1

domain-needed
bogus-priv

cache-size=1000

local=/lab/

address=/esp32.lab/192.168.10.2
address=/router.lab/192.168.10.1
address=/vm10.lab/192.168.10.100
```

---

## Explicação da configuração

| Diretiva | Função |
| :--- | :--- |
| `no-resolv` | ignora DNS padrão do sistema |
| `server=` | define DNS upstream |
| `domain-needed` | evita consultas inválidas |
| `bogus-priv` | filtra respostas privadas inválidas |
| `cache-size` | habilita cache DNS |
| `local=/home/` | domínio local interno |
| `address=` | cria registros DNS locais |

---

## Reinicialização do serviço

Após alterar a configuração:

```bash
sudo systemctl restart dnsmasq
```

Verificação do status:

```bash
sudo systemctl status dnsmasq
```

---

# Configuração do TR1200

## Objetivo

O roteador continuou utilizando o DNS padrão do ISP para acesso à internet, porém passou a entregar via DHCP:

- DNS primário → VM Debian
- DNS alternativo → próprio roteador

Isso permite:

- resolução local via dnsmasq
- fallback automático caso a VM esteja desligada
- manutenção do acesso à internet mesmo sem o servidor DNS local

---

## Configuração via LuCI

Menu utilizado:

```txt
Network → Interfaces → LAN → DHCP Server
```

Configuração aplicada:

| Ordem | DNS |
| :--- | ---: |
| DNS 1 | 192.168.10.100 |
| DNS 2 | 192.168.10.1 |

---

## Fluxo de funcionamento

### VM online

```txt
cliente → VM Debian → Google DNS
```

### VM offline

```txt
cliente → roteador → DNS ISP/Google
```

---

## Testes realizados

### Teste de resolução DNS

```bash
ping esp32.lab
```

Resultado:

```txt
PING esp32.lab (192.168.10.2)
```

---

### Teste HTTP

```bash
curl http://esp32.lab
```

---

### Teste em navegador

```txt
http://esp32.lab
http://router.lab
http://vm10.lab
```

---

# Conceitos estudados

- DNS local
- dnsmasq
- resolução de nomes
- upstream DNS
- DHCP
- fallback DNS
- infraestrutura Linux
- homelab
- service discovery
- redes LAN

---

# Resultado final do laboratório

Recursos funcionando:

- DNS local na LAN
- resolução de nomes amigáveis
- fallback DNS automático
- cache DNS local
- integração Debian + OpenWrt
- acesso simplificado aos serviços internos
- infraestrutura DNS funcional para homelab
- monitoramento em tempo real (sudo journalctl -u dnsmasq -f)

---

# Considerações finais

Este laboratório expandiu a infraestrutura do laboratório anterior baseado em VPN e serviços internos, adicionando uma camada de resolução de nomes semelhante a ambientes reais de rede.

A utilização do dnsmasq em uma VM Debian permitiu implementar um servidor DNS simples, leve e funcional, integrado ao DHCP do roteador OpenWrt/CudyOS.

Além da praticidade de acesso aos dispositivos da rede, o laboratório serviu como introdução prática aos conceitos de DNS interno, forwarding, cache e infraestrutura de serviços em Linux.