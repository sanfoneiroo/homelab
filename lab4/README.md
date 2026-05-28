# LAB4 — Servidor de Mídia com CasaOS + Jellyfin + yt-dlp

## 1 Introdução

Neste laboratório foi criada uma VM Debian dedicada a serviços multimídia e automação doméstica leve, utilizando:

* CasaOS como painel de gerenciamento self-hosted
* Jellyfin como servidor de mídia
* yt-dlp para download e conversão de conteúdo online
* ffmpeg para processamento de áudio/vídeo

O objetivo do ambiente é centralizar arquivos multimídia dentro do homelab, permitindo:

* streaming local
* organização de bibliotecas
* acesso remoto via VPN
* gerenciamento simplificado via interface web
* experimentação com aplicações Docker/self-hosted

---

# 2 Estrutura Geral do Ambiente

## Máquina Virtual

Sistema operacional:

* Debian Linux

Funções principais:

* CasaOS
* Jellyfin
* yt-dlp
* ffmpeg

---

# 3 Instalação do CasaOS

O CasaOS foi instalado diretamente via script oficial.

## Instalação

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

## Inicialização manual

```bash
sudo systemctl start casaos
```

Após a instalação, o painel web fica acessível pela rede local através do navegador.

Exemplo:

```text
http://IP_DA_VM:81
```

---

# 4 Jellyfin

O Jellyfin foi instalado diretamente pela interface gráfica do CasaOS.

## Processo

* abrir App Store do CasaOS
* buscar por Jellyfin
* instalar container
* aguardar deploy automático

O CasaOS realiza automaticamente:

* download da imagem Docker
* criação do container
* exposição da interface web
* gerenciamento básico do serviço

---

# 5 Organização de Diretórios

Foi utilizada uma separação simples entre:

## Arquivos gerais da VM

```text
/home/user
```

## Biblioteca de mídia do Jellyfin

```text
/data/media
```

Estrutura sugerida:

```text
/data/media
├── filmes
├── series
├── musicas
└── downloads
```

Essa separação facilita:

* backup
* montagem de volumes Docker
* expansão futura
* organização do storage

---

# 6 Configuração Inicial do Jellyfin

Após acessar a interface web:

## Criar servidor

Etapas:

* definir usuário administrador
* configurar idioma
* selecionar diretórios de mídia

## Adicionar biblioteca

Exemplo:

```text
/data/media/musicas
```

Tipos de biblioteca:

* Filmes
* Séries
* Música
* Vídeos

O Jellyfin realiza automaticamente:

* varredura dos diretórios
* geração de metadata
* download de capas
* indexação da biblioteca

---

# 7 Instalação do yt-dlp

O yt-dlp foi utilizado para download de áudio diretamente do YouTube.

Este laboratório possui finalidade exclusivamente educacional e experimental dentro do contexto de homelab e automação Linux.

Recomenda-se utilizar a ferramenta apenas para conteúdo autoral, mídias livres, arquivos com permissão de download ou material disponibilizado legalmente pelos próprios criadores

## Dependência ffmpeg

```bash
sudo apt install ffmpeg
```

## Download do yt-dlp

```bash
sudo curl -L https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp \
  -o /usr/local/bin/yt-dlp
```

## Permissão de execução

```bash
sudo chmod a+rx /usr/local/bin/yt-dlp
```

---

# 8 Download e Conversão para MP3

## Exemplo básico

```bash
yt-dlp -f bestaudio \
  --extract-audio \
  --audio-format mp3 \
  --audio-quality 0 \
  "URL_DO_VIDEO"
```

O processo realiza:

1. download do melhor áudio disponível
2. extração do stream de áudio
3. conversão para MP3
4. processamento via ffmpeg

---

# 9 Observações Técnicas

## CasaOS

O CasaOS atua principalmente como:

* painel web
* gerenciador Docker simplificado
* gateway para aplicações self-hosted

Na prática, os containers continuam sendo executados pelo Docker instalado no sistema.

## Jellyfin

O Jellyfin funciona como alternativa self-hosted ao:

* Plex
* Netflix
* Spotify local

Com suporte a:

* streaming LAN
* transcodificação
* metadata automática
* múltiplos usuários

## yt-dlp

O yt-dlp é extremamente útil em laboratórios para:

* arquivamento pessoal
* testes multimídia
* automação de downloads
* integração com scripts

---

# 10 Conclusão

Este laboratório consolidou:

* uso de Docker em ambiente real
* self-hosting multimídia
* gerenciamento web simplificado
* organização de storage
* automação de mídia no Linux

Além do aspecto técnico, o ambiente se mostrou extremamente prático e divertido de utilizar dentro do homelab.
