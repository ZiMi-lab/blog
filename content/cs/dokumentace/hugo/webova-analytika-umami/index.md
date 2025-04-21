+++
date = '2025-04-21T09:42:10Z'
draft = false
title = 'Vlastní webová analytika s Umami a Cloudflare Tunnel pro maximální soukromí'
tags = ['Umami', 'Docker', 'Cloudflare Tunnel', 'Hugo', 'Home Lab']
kategorie = ['Návod']
ShowToc = true
TocOpen = true
comments = true
+++
V dnešní době je soukromí na internetu velké téma. Pokud provozujete webové stránky, pravděpodobně chcete vědět, kolik lidí je navštěvuje, odkud přicházejí, jaký obsah je nejpopulárnější. Google Analytics je často používané řešení, ale co když nechcete svá data svěřovat velké korporaci? Co když chcete mít plnou kontrolu nad analytickými daty a zajistit, že nebudou zneužita pro cílení reklam nebo sledování uživatelů?

Přesně to byl můj případ. Zkouším open-source nástroj na vlastní infrastruktuře. Vybral jsem **Umami** a **Cloudflare Tunnel** pro bezpečné zpřístupnění světu. V tomto článku vás provedu celým procesem.

## Co je Umami?

[Umami](https://umami.is/) je moderní, jednoduchý a open-source nástroj pro webovou analytiku. Jeho hlavními výhodami jsou:

1. **Zaměření na soukromí:** Nesbírá žádné osobní údaje, nepoužívá cookies a je plně v souladu s GDPR, ... Není potřeba žádný otravný cookie banner jen kvůli analytice.
2. **Vlastnictví dat:** Protože si Umami hostuji sám, mám 100% kontrolu nad svými daty.
3. **Jednoduchost:** Nabízí přehledné rozhraní a poskytuje všechny klíčové metriky bez zbytečné komplexity.
4. **Nízká zátěž:** Je napsaný v Node.js a je velmi lehký a rychlý. Sledovací skript má jen pár kilobajtů.
5. **Open Source:** Můžete si kód prohlédnout, upravit a přispět jeho rozvoji.

## Než začneme, budete potřebovat

- Server nebo virtuální stroj (např. v rámci Home Labu), kde běží **Docker**.
- Nainstalovaný **Portainer** (volitelné, ale usnadňuje správu Docker kontejnerů – v tomto návodu jej používám).
- Účet u **Cloudflare** a doménu spravovanou přes Cloudflare (postačí DNS zóna).
- Základní znalost práce s příkazovou řádkou a Dockerem.

## Krok 1: Nasazení Umami (Docker Compose a Portaineru)

{{< resized-img src="Portainer_Stack_umami.png" size="800x jpg" alt="Partainer: Add stack" >}}

Pro správu Docker kontejnerů aktuálně zkouším Portainer.

1. Přihlaste se do Portaineru.
2. Přejděte do sekce **Stacks**.
3. Klikněte na **+ Add stack**.
4. Pojmenujte stack, například `umami`.
5. Do **Web editoru** vložte následující `docker-compose.yml` konfiguraci:

```
services:
  umami:
    image: ghcr.io/umami-software/umami:postgresql-latest
    ports:
      - "3000:3000" # Můžete změnit vnitřní port, pokud 3000 již používáte
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
      DATABASE_TYPE: postgresql
      APP_SECRET: ${APP_SECRET} # Důležité pro bezpečnost session!
    depends_on:
      db:
        condition: service_healthy
    init: true
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:3000/api/heartbeat || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - umami_db_data:/var/lib/postgresql/data
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  umami_db_data:
```

6. Nyní je potřeba nastavit **Environment variables** (proměnné prostředí) pro tento stack v Portaineru:
    
    - `POSTGRES_DB`: Zvolte název databáze pro Umami (např. `umami_db`).
    - `POSTGRES_USER`: Zvolte uživatelské jméno pro přístup k databázi (např. `umami_user`).
    - `POSTGRES_PASSWORD`: Vygenerujte **silné a bezpečné heslo** pro databázového uživatele.
    - `APP_SECRET`: Vygenerujte náhodný řetězec (salt), který Umami používá pro zabezpečení. Můžete použít příkaz v terminálu:
        ```
        openssl rand -base64 32
        ```
        
        Výstup zkopírujte a vložte jako hodnotu pro `APP_SECRET`.
7. Klikněte na **Deploy the stack**. Portainer stáhne potřebné image a spustí kontejnery pro Umami a PostgreSQL databázi.    

Po úspěšném spuštění by měla být aplikace Umami dostupná na adrese vašeho serveru na portu 3000 (např. `http://<IP_ADRESA_SERVERU>:3000`).

### První přihlášení a zabezpečení

Výchozí přihlašovací údaje jsou:

- Uživatelské jméno: `admin`
- Heslo: `umami`

**Důležité:** Po prvním přihlášení si **okamžitě změňte heslo** na silné a unikátní v nastavení profilu!

## Krok 2: Vytvoření Cloudflare Zero Trust Tunnel

{{< resized-img src="CF_create_tunnel.png" size="800x jpg" alt="Cloudflare: Create tunnel" >}}

Aby byla instance Umami bezpečně dostupná z internetu (pro sběr dat z webů a pro přístup k dashboardu) a abychom nemuseli otevírat porty na firewallu, zvěřejnit svou IP adresu, použijeme Cloudflare Tunnel.

1. Přihlaste se do [Cloudflare Dashboard](https://dash.cloudflare.com/).
2. Vyberte doménu a přejděte do sekce **Zero Trust** (pokud ji nemáte aktivovanou, Cloudflare vás provede rychlým nastavením plánu).
3. V Zero Trust dashboardu jděte do **Networks** -> **Tunnels**.
4. Klikněte na **+ Create a tunnel**.
5. Vyberte typ konektoru **Cloudflared**. Klikněte na **Next**.
6. Zadejte **název tunelu** (např. `umami-tunnel`). Klikněte na **Save Tunnel**.
7. Nyní Cloudflare zobrazí příkazy pro instalaci `cloudflared` konektoru. Protože jej spouštím v Dockeru, vyberu záložku **Docker**. Zobrazí se příkaz obsahující unikátní **token** tunelu. Tento token si **zkopíruji a bezpečně uložím** (za chvíli bude potřeba).
```
docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token <TOKEN>
```
8. V sekci **Public Hostnames** klikněte na **+ Add public hostname**:
    - **Subdomain:** Subdoména, na které bude Umami dostupné (např. `stats` nebo `umami`).
    - **Domain:** Vyberte doménu ze seznamu. Výsledná adresa bude např. `stats.vasedomena.cz`.
    - **Service - Type:** Vyberte `HTTP`.
    - **Service - URL:** Zadejte `host.docker.internal:3000`.
        - _Poznámka:_ `host.docker.internal` je speciální DNS název v rámci Dockeru, který umožňuje kontejneru (`cloudflared`) komunikovat se službou běžící na hostitelském stroji (nebo v jiném kontejneru vystaveném na hostitelském portu). Port `3000` je ten, který jsme nastavili pro Umami v předchozím kroku.
9. Klikněte na **Save tunnel**. Tím je tunel nakonfigurován na straně Cloudflare.

## Krok 3: Spuštění Cloudflared kontejneru v Portaineru

Nyní vytvoříme další stack v Portaineru pro `cloudflared` konektor, který se připojí k Cloudflare a bude směrovat provoz.

1. V Portaineru jděte opět do **Stacks** -> **+ Add stack**.
2. Pojmenujte stack, například `cloudflared`.
3. Vložte následující `docker-compose.yml` obsah:

```
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    environment:
      - TZ=Europe/Prague # Nastavte časovou zónu
      - TUNNEL_TOKEN=${TUNNEL_TOKEN} # Zde vložíme token z Cloudflare
    command: tunnel --no-autoupdate run
    # Tato sekce je důležitá, aby cloudflared viděl služby na hostiteli
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

4. V sekci **Environment variables** přidejte proměnnou: `TUNNEL_TOKEN`: Vložte sem ten dlouhý **token**, který jsme získali při vytváření tunelu v Cloudflare dashboardu.
5. Nastavte správnou **časovou zónu (TZ)**.
6. Klikněte na **Deploy the stack**.

Pokud vše proběhlo v pořádku, kontejner `cloudflared` se spustí, připojí se k Cloudflare pomocí vašeho tokenu a váš tunel bude aktivní.

## Krok 4: Přidání webu do Umami a získání Tracking kódu

{{< resized-img src="Umami_detail.png" size="800x jpg" alt="Umami: Nastavení" >}}

Nyní zaregistrujeme web v Umami instanci.

1. Otevřete Umami instanci přes Cloudflare Tunnel adresu (např. `https://stats.vasedomena.cz`) a přihlaste se.
2. Přejděte do **Settings** -> **Websites** -> **+ Add website**.
3. Vyplňte **Název (Name)** webu (libovolný, pro snadnou identifikaci) a **Doménu (Domain)** webu (např. `www.mojedomena.cz`).
4. Klikněte na **Save**.
5. Po uložení se web objeví v seznamu. Klikněte na `Upravit` a zobrazí se Vám nastavení, kde uvidíte `Website ID` a na vedlejší záložce i celý sledovací kód. Tento kód budete potřebovat v dalším kroku. Všimněte si dvou klíčových částí:
    - Adresa skriptu (atribut `src`): Například `https://stats.vasedomena.cz/script.js`.
    - Website ID (atribut `data-website-id`): Unikátní identifikátor vašeho webu v Umami (dlouhý řetězec písmen a čísel).

## Krok 5: Integrace Tracking kódu do webu (příklad pro Hugo)

Pokud web běží na generátoru [Hugo](https://gohugo.io/), stejně jako můj blog, můžete sledovací kód integrovat elegantně pomocí konfigurace a partial šablony.

### Konfigurace v `config.yaml` (nebo `config.toml`)

Otevřete hlavní konfigurační soubor Hugo projektu (`config.yaml`, `hugo.yaml` nebo `config.toml` - syntaxe se mírně liší, zde příklad pro YAML). Do sekce `params` přidejte následující blok:

```
params:
  # ... vaše ostatní parametry ...
  umami:
    script: "https://stats.vasedomena.cz/script.js"  # Nahraďte vaší URL Umami skriptu
    websiteID: "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" # Nahraďte vaším skutečným Website ID z Umami
  # ... další parametry ...
```

**Důležité:** Nahraďte URL v `script` adresou vašeho Umami skriptu a `websiteID` vaším unikátním identifikátorem získaným v kroku 4.

### Vytvoření Partial šablony pro vložení do `<head>`

Mnoho Hugo témat umožňuje rozšířit HTML hlavičku (`<head>`) pomocí partial šablony. Standardní název pro takovou šablonu bývá `extend_head.html`.

1. Ve struktuře vašeho Hugo projektu vytvořte (pokud neexistuje) soubor:
    `layouts/partials/extend_head.html`
2. Do tohoto souboru vložte následující řádek:
    ```
    {{/* Umami Tracking Code - loaded if configured in params */}}
    {{ if and .Site.Params.umami.script .Site.Params.umami.websiteID }}
      <script defer src="{{ .Site.Params.umami.script }}" data-website-id="{{ .Site.Params.umami.websiteID }}"></script>
    {{ end }}
    ```
    
    - Použití `defer` zajistí, že se skript načte asynchronně a nebude blokovat vykreslování stránky.
    - Podmínka `{{ if ... }}` zajistí, že se skript vloží pouze tehdy, pokud jsou v `config.yaml` vyplněny obě hodnoty (`script` a `websiteID`).


    **Poznámka:** Ujistěte se, že vaše Hugo téma skutečně volá partial `extend_head.html` ve své hlavní šabloně hlavičky (obvykle v `layouts/_default/baseof.html` nebo podobném souboru, hledejte něco jako `{{ partial "extend_head.html" . }}`). Pokud ne, budete muset výše uvedený `<script>` tag vložit přímo do příslušné šablony vašeho tématu v rámci `<head>` tagu.

### Nasazení změn

1. Znovu sestavte svůj Hugo web pomocí příkazu `hugo` v terminálu.
2. Nasaďte vygenerované soubory (obvykle z adresáře `public`) na váš webhosting.

## Krok 6: Ověření a závěr

Po nasazení změn na webu otevřete web v prohlížeči (ideálně v anonymním okně, abyste neměli cachovaný obsah). Můžete také použít vývojářské nástroje prohlížeče (klávesa F12) a na záložce "Network" (Síť) zkontrolovat, zda se úspěšně načítá váš Umami `script.js`.

Pokud je vše OK, ihned uvidíte první data o návštěvnosti webu na Umami dashboardu.

Úspěšně jsme nastavili vlastní, na soukromí zaměřenou webovou analytiku pomocí Umami, Dockeru a Cloudflare Tunnel. Máme data o návštěvnosti webu, aniž bychom je museli sdílet s třetími stranami.