# LAB 8 — Inicialização Automática de Máquina Virtual Debian no macOS

## 1. Objetivo

O objetivo deste laboratório é configurar o macOS para iniciar automaticamente uma máquina virtual Debian durante o processo de boot, antes mesmo do login do usuário, evitando o processo de:

1. Fazer login no macOS;
2. Abrir o VirtualBox;
3. Iniciar a máquina virtual manualmente.

---

## 2. Conceitos Utilizados

O macOS utiliza o sistema **launchd** para gerenciamento de serviços.

Os serviços são definidos através de arquivos:

- LaunchAgents (executados após login)
- LaunchDaemons (executados durante a inicialização do sistema)

Neste laboratório será utilizado um **LaunchDaemon**, permitindo que o VirtualBox execute a máquina virtual logo após o carregamento do sistema operacional.

Fluxo simplificado:

```text
Mac ligado
      │
      ▼
macOS inicia
      │
      ▼
launchd carrega LaunchDaemons
      │
      ▼
VBoxManage executa
      │
      ▼
Debian inicia em modo headless
      │
      ▼
Serviços ficam disponíveis na rede
```

---

## 3. Pré-requisitos

- macOS
- VirtualBox instalado
- Máquina virtual Debian funcional
- Acesso administrativo ao sistema

Antes de prosseguir, confirme o nome exato da VM:

```bash
VBoxManage list vms
```

Exemplo:

```bash
"Debian_Server" {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}
```

---

## 4. Criação do LaunchDaemon

Criar o arquivo:

```bash
sudo nano /Library/LaunchDaemons/com.vbox.vm.Debian_Server.plist
```

Inserir:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">

<plist version="1.0">
<dict>

    <key>Label</key>
    <string>com.vbox.vm.Debian_Server</string>

    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/VBoxManage</string>
        <string>startvm</string>
        <string>Debian_Server</string>
        <string>--type</string>
        <string>headless</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>KeepAlive</key>
    <false/>

    <key>UserName</key>
    <string>usuario</string>

</dict>
</plist>
```

---

## 5. Configuração de Permissões

Definir proprietário:

```bash
sudo chown root:wheel \
/Library/LaunchDaemons/com.vbox.vm.Debian_Server.plist
```

Definir permissões:

```bash
sudo chmod 644 \
/Library/LaunchDaemons/com.vbox.vm.Debian_Server.plist
```

---

## 6. Carregamento do Serviço

Ativar o LaunchDaemon:

```bash
sudo launchctl load -w \
/Library/LaunchDaemons/com.vbox.vm.Debian_Server.plist
```

Verificar se foi carregado:

```bash
sudo launchctl list | grep vbox
```

---

## 7. Testes

Reiniciar o computador:

```bash
sudo reboot
```

Após o boot, sem abrir o VirtualBox, verificar:

```bash
VBoxManage list runningvms
```

Resultado esperado:

```bash
"Debian_Server" {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}
```

Também é possível testar acessando os serviços da VM pela rede:

```bash
ssh usuario@IP_DA_VM
```

---

## 8. Arquitetura Resultante

```text
┌─────────────────────────────┐
│        Mac Mini Host        │
│                             │
│  macOS                      │
│  LaunchDaemon               │
│  VirtualBox                 │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│      Debian Server VM       │
│                             │
│  SSH                        │
│  Docker                     │
│  VPN                        │
│  Servidor Web               │
│  Serviços de Rede           │
└─────────────────────────────┘
```

---

## 9. Vantagens Obtidas

### Maior disponibilidade

Após quedas de energia ou reinicializações, o servidor retorna automaticamente.

### Operação sem monitor

A VM pode funcionar totalmente em modo headless.

### Menor intervenção manual

Não é necessário:

- Abrir VirtualBox;
- Fazer login;
- Iniciar a VM manualmente.

### Comportamento semelhante a um servidor dedicado

O conjunto Mac + Debian passa a operar de forma próxima a um servidor físico tradicional.

---

## 10. Considerações Importantes

O caminho do `VBoxManage` pode variar conforme a versão do VirtualBox.

Verifique com:

```bash
which VBoxManage
```

Em algumas instalações recentes ele pode estar localizado em:

```bash
/Applications/VirtualBox.app/Contents/MacOS/VBoxManage
```

Caso o daemon não funcione após reinicialização, esse é um dos primeiros itens a verificar.

Também é recomendável testar a inicialização manual da VM em modo headless:

```bash
VBoxManage startvm "Debian_Server" --type headless
```

Isso ajuda a validar o funcionamento antes de configurar a automação.

---

## Conclusão

A utilização de um LaunchDaemon permitiu transformar uma máquina virtual Debian hospedada no VirtualBox em um serviço iniciado automaticamente durante o boot do macOS. Com isso, os serviços de rede permanecem disponíveis mesmo após reinicializações do host, aproximando o ambiente de homelab do comportamento esperado em servidores dedicados e aumentando significativamente a disponibilidade da infraestrutura.

Como próximos passos, podem ser implementados mecanismos de monitoramento e recuperação automática da VM, permitindo que o próprio macOS detecte falhas e tente reiniciar o ambiente virtualizado de forma transparente ao usuário.