# 📘 Blog – Statický obsah generovaný pomocí Hugo + PaperMod

Tento repozitář obsahuje statický obsah osobního blogu zaměřeného na **chytrou domácnost**, **Home Lab**, **kybernetickou bezpečnost** a **umělou inteligenci**.  
Blog je generován pomocí [Hugo](https://gohugo.io/) s šablonou [PaperMod](https://github.com/adityatelange/hugo-PaperMod) a je nasazen přes [Cloudflare Pages](https://pages.cloudflare.com/).

🌐 Online verze blogu: **[https://zimacek.cz](https://zimacek.cz)**

---

## 🧠 Témata, kterým se blog věnuje

- **Smart Home:** Home Assistant, Zigbee/Matter, automatizace domácnosti
- **Home Lab:** Proxmox, kontejnery, NAS, vlastní serverová řešení
- **Sítě:** VLAN, Wi-Fi, firewall, segmentace domácích sítí
- **Kybernetická bezpečnost:** ochrana dat, hrozby, zabezpečení IoT
- **Umělá inteligence:** lokální jazykové modely, AI agenti, zpracování dat

---

## 📁 Struktura repozitáře
```
/content/       → články v Markdownu
/assets/        → obrázky, diagramy, vizuály
/layouts/       → přizpůsobení šablony PaperMod
/static/        → veřejné statické soubory
/config.yaml    → hlavní konfigurační soubor Hugo
/.env.example   → vzorový soubor s proměnnými prostředí
```

---

## 🔧 Proměnné prostředí

Pro správný běh některých funkcí můžeš využít soubor `.env` – ten ale nikdy necommituj.  
Místo toho použij `.env.example` jako šablonu a zkopíruj jej:

```bash
cp .env.example .env
```

Ukázka obsahu `.env.example`:

```env
# Google Analytics
GOOGLE_ANALYTICS_ID=G-XXXXXXXXXX

# API nebo služba pro komentáře
DISQUS_SHORTNAME=your-disqus-shortname

# Základní adresa blogu
BASE_URL=https://zimacek.cz
```

---

## 🛠️ Cloudflare Pages – Build nastavení

Při nasazení blogu přes [Cloudflare Pages](https://pages.cloudflare.com/) nastav ve svém projektu následující parametry:

| Parametr | Hodnota |
|----------|---------|
| **Framework preset** | `None` |
| **Build command** | `hugo` |
| **Build output directory** | `public` |

Pro zajištění správné verze Hugo nastav také environment proměnnou:

```
Key:    HUGO_VERSION  
Value:  0.145.0
```

Tím zajistíš, že Cloudflare použije stejnou verzi Hugo jako máš při lokálním vývoji (`hugo v0.145.0+extended`).

---

## 🖼️ Práce s obrázky a automatická změna velikosti

Pokud chceš v článcích používat obrázky s automatickou změnou velikosti pomocí shortcode `resized-img`, je potřeba dodržet určitou strukturu obsahu.

---

✅ **Řešení**

### ✅ Převést článek na *Page Bundle*

1. Vytvoř složku s názvem článku:

```
content/cs/dokumentace/chytra-domacnost/mereni-spotreby-vody/
```

2. Přesuň Markdown soubor do této složky a přejmenuj ho na `index.md`  
3. Vlož obrázek `obrazek.png` do stejné složky  
4. V Markdownu použij následující shortcode:

```markdown
{{< resized-img src="obrazek.png" size="600x jpg" alt="Obrázek" >}}
```

---

### ✅ Vlastní shortcode

Soubor je uložen v `layouts/shortcodes/resized-img.html`

```go-html-template
{{ $img := .Page.Resources.GetMatch (.Get "src") }}
{{ if $img }}
  {{ $resized := $img.Resize (.Get "size") }}
  <img src="{{ $resized.RelPermalink }}" alt="{{ .Get "alt" }}">
{{ end }}
```

## 🔐 Bezpečnost a konfigurace

- `.env` soubor je zahrnutý v `.gitignore`, aby se **nikdy nedostal do repozitáře**.
- Nepřidávej do verzování žádné soukromé klíče, tokeny nebo interní URL.
- Doporučuji oddělit drafty (koncepty článků) do samostatného privátního repozitáře nebo složky mimo `public`.

📝 Obsah tohoto repozitáře je dostupný pod licencí [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

#SmartHome #HomeLab #CyberSecurity #UměláInteligence #Blog #Hugo #CloudflarePages