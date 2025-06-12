+++
date = '2025-03-20T10:32:18Z'
draft = false
title = 'Jak nasadit Hugo blog na GitHub a Cloudflare Pages'
kategorie = ['Návod']
tags = ['Cloudflare', 'Git', 'Hugo']
ShowToc = true
TocOpen = true
comments = true
+++

Pokud chcete provozovat moderní a rychlý blog bez nutnosti správy serveru, kombinace **Hugo**, **GitHub** a **Cloudflare Pages** je skvělou volbou. V tomto článku projdeme krok za krokem:

## ⚙️ 1. Instalace Hugo projektu

Nainstalujte Hugo Extended verzi, např. [podle návodu]({{< relref "dokumentace/hugo/provoz-instalace.md" >}}).

Soubor `config.yaml` si můžete nastavit např. podle [mého vzorového, který je umístěn na GitHubu](https://github.com/ZiMi-lab/blog).

## 🗂️ 2. Vytvoření GitHub repozitáře

Začněte vytvořením nového repozitáře na GitHubu:

1. Přejděte na [github.com/new](https://github.com/new)
2. Vyplňte:
    - **Název**: např. `blog`
    - Popis: např. `Statický obsah blogu generovaného pomocí Hugo`
    - Viditelnost: `Public` (nebo `Private`, pokud preferujete)
3. Vytvořte repozitář bez README, `.gitignore` a licence (přidáme ručně později).

Na počítači a ve složce, kde máte blog umístěn, spusťte inicializaci a připojení ke vzdálenému repozitáři:
```bash
git init
git remote add origin https://github.com/<uzivatel>/<nazev-repozitare>.git
```

## 🏗️ 3. Struktura složek Hugo

Hugo používá tuto základní strukturu:
```
/content/       → články v Markdownu
/assets/        → obrázky, diagramy, vizuály
/layouts/       → přizpůsobení šablony PaperMod
/static/        → veřejné statické soubory
/config.yaml    → hlavní konfigurační soubor Hugo
/.env.example   → vzorový soubor s proměnnými prostředí (zatím není potřeba)
```

Pro první článek (mrkněte do [oficiální dokumentace, jak tvořit obsah](https://gohugo.io/documentation/)):

Otevřete `content/posts/prvni-clanek.md` a nastavte `draft: false`, jinak se článek nezobrazí.

## ☁️ 4. Propojení s Cloudflare Pages

1. Přejděte na [pages.cloudflare.com](https://pages.cloudflare.com/)
2. Klikněte na **"Create a Project"**
3. Připojte GitHub účet a vyberte repozitář
4. Nastavte:
    - **Framework preset**: `None`
    - **Build command**: `hugo`
    - **Build output directory**: `public`

Volitelně nastavte verzi Hugo, kterou používáte:

Cloudflare automaticky spustí build a nasadí blog. Po úspěšném buildu získáte veřejnou adresu (např. `https://nazev-projektu.pages.dev`) a můžete připojit vlastní doménu.

## 🧪 Tipy pro ladění

- Pokud je blog prázdný, ujistěte se, že máte `content/_index.md` nebo aktivní články s `draft: false`
- Sledujte build log v Cloudflare Pages – často pomůže rychle odhalit chybu
- `.env` soubor nepřidávejte do gitu, použijte `.env.example`

## 🔗 Odkazy a zdroje

- [Provoz a instalace Hugo v LXC kontejneru]({{< relref "dokumentace/hugo/provoz-instalace.md" >}})
- [Hugo: Dokumentace](https://gohugo.io/documentation/)
- [Příklad repozitáře](https://github.com/ZiMi-lab/blog)

---

Chcete si postavit vlastní blog bez serveru? S GitHubem, Hugem a Cloudflare Pages je to otázka pár minut. 📘