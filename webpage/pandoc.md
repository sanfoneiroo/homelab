# Webpage Tools

Este diretório contém ferramentas e recursos utilizados para gerar páginas HTML estáticas a partir de arquivos Markdown (.md), mantendo uma renderização semelhante ao GitHub em modo escuro.

O objetivo é facilitar a criação de documentação técnica leve para hospedagem local no ESP32 File Server.

---

# Ferramenta utilizada

## Pandoc

Site oficial:

https://pandoc.org

---

# Instalação no macOS

## Instalação via Homebrew

```bash
brew install pandoc
```

---

## Verificação da instalação

```bash
pandoc --version
```

Resultado esperado:

```txt
pandoc 3.x.x
```

---

# Conversão Markdown → HTML


```bash
pandoc file.md -s --toc -o file.html
```

---

# Explicação dos parâmetros

| Opção | Função |
| :--- | :--- |
| `-s` | gera HTML completo |
| `--toc` | cria índice automático |
| `-o` | define arquivo de saída |

---

# Teste local

## Abrir diretamente no navegador

```bash
open lab1.html
```

---

# Fluxo utilizado

```txt
Markdown (.md)
        ↓
Pandoc
        ↓
HTML estático
        ↓
Upload para ESP32
        ↓
Portal técnico local
```

---

# Resultado

O HTML gerado possui:

- suporte a imagens locais
- baixo consumo de recursos
- compatibilidade com ESP32 File Server