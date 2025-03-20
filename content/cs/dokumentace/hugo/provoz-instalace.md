+++
date = '2025-02-27T12:23:15Z'
title = 'Instalace a provoz Huga v LXC kontejneru na Proxmoxu'
kategorie = ['Návod']
tags = ['Git', 'Go', 'Hugo']
ShowToc = true
TocOpen = true
+++

Tento návod popisuje instalaci **Huga** v **extended** edici v LXC kontejneru s Ubuntu na Proxmoxu. Instalace probíhá ze zdrojových kódů.

## 🔧 Požadavky

Než začneme, ujistěte se, že máte nainstalované následující komponenty:

- **Git** – pro získání zdrojových kódů
- **Go (verze 1.24 nebo novější)** – nutné pro kompilaci
- **GCC** – překladač potřebný pro kompilaci Go programů

## 📦 Instalace

### 1. Instalace Git a GCC

Nejprve nainstalujte **Git** a **GCC**:

```bash
apt update && apt install -y git-all gcc g++
```

### 2. Instalace Go

Nainstalujte **Go** podle [oficiální příručky](https://go.dev/doc/install). Po instalaci ověřte verzi:

```bash
go version
```

### 3. Instalace Hugo ze zdrojových kódů

**Extended edice** Huga je nutná pro správnou transpilaci **Sass do CSS** a minifikaci CSS a JS souborů.

Spusťte následující příkaz pro instalaci Huga:

```bash
CGO_ENABLED=1 go install -tags extended github.com/gohugoio/hugo@latest
```

Tímto se Hugo nainstaluje do výchozího adresáře `$HOME/go/bin`.

> **Poznámka:** Minifikace odstraňuje nepotřebné znaky (mezery, komentáře, nové řádky) a zrychluje načítání webu.

### 4. Aktualizace proměnné prostředí `PATH`

Aby bylo možné spouštět Hugo z příkazové řádky, je nutné přidat jeho umístění do systémové proměnné `PATH`.

Pro **Bash** (Ubuntu, Debian):

```bash
echo 'export PATH="$HOME/go/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Pro **Zsh** (macOS, některé distribuce Linuxu):

```bash
echo 'export PATH="$HOME/go/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

## 🐚 Jak zjistit, který shell používám?

V Linuxu i na macOS stačí zadat:

```bash
echo $SHELL
```

---

Tento návod by vám měl pomoci s instalací a konfigurací Huga v LXC kontejneru na Proxmoxu. Pokud narazíte na problémy, ověřte si správnost instalace jednotlivých závislostí nebo se podívejte na [oficiální dokumentaci](https://gohugo.io/).
