+++
date = '2025-06-06T07:30:00Z'
draft = false
title = 'Proxmox, Gmail a notifikace: Průvodce krok za krokem'
description = 'Trápí vás notifikace z Proxmoxu? Ukážeme si, jak nakonfigurovat SMTP s Gmailem krok za krokem.'
kategorie = ['návod']
tags = ['Proxmox', 'Gmail', 'notifikace']
ShowToc = true
TocOpen = true
comments = true
+++

Podíváme se na něco, co vnímám jako klíčovou součást monitoringu každého **Home Labu** – na **notifikace**. Proxmox ve výchozím nastavení obsahuje lokální e-mailový server **Postfix**, ale jeho použití pro zasílání notifikací do internetu není ideální volbou. Ukážeme si, jak jej elegantně nahradit odesíláním přes **SMTP servery** od Googlu.

Proč je to důležité? Představte si, že vám selže disk, spadne virtuální stroj nebo dojde místo na úložišti. Pokud nemáte nastavené notifikace a nesledujete stav serveru 24/7, můžete se o problému dozvědět až ve chvíli, kdy je pozdě. V tomto článku si krok za krokem ukážeme, jak nastavit Proxmox tak, aby vám spolehlivě posílal e-maily přes Gmail, i když Proxmox jako takový nepodporuje přihlášení přes OAuth.

## Proč nepoužívat výchozí Postfix pro notifikace z Proxmoxu?

Proxmox ve výchozím nastavení spoléhá pro systémové notifikace na lokálně nainstalovaný **Postfix**. Jedná se o standardní a robustní MTA (Mail Transfer Agent), který je výchozí v Debianu, na kterém Proxmox běží. Na první pohled to vypadá jako "hotové řešení", ale má to svá úskalí, která se pro jednoduché odesílání notifikací vyplatí znát:

1. **Zbytečná složitost konfigurace:** Správné nastavení pro odesílání přes externí server (tzv. relay) není pro začátečníka triviální. Vyžaduje to úpravu konfiguračních souborů jako, vytvoření souboru s heslem a znalost direktiv pro autentizaci (SASL) a šifrování (TLS).
2. **Spam a reputace:** Toto je klíčový problém. Pokud lokální Postfix odesílá e-maily přímo z vaší domácí nebo serverové IP adresy, je téměř jisté, že vaše notifikace skončí ve spamu. Poskytovatelé e-mailu (Gmail, Seznam, atd.) nedůvěřují zprávám z IP adres bez správné konfigurace DNS záznamů (jako jsou **SPF**, **DKIM** a **DMARC**). Důležité upozornění na selhání disku by vám tak vůbec nemuselo dorazit.
3. **Chybějící autentizace a šifrování (bez další konfigurace):** Výchozí nastavení Postfixu slouží pro lokální doručování. Aby mohl bezpečně odesílat notifikace do internetu, musí se explicitně nakonfigurovat pro použití šifrování (TLS/SSL) a silné autentizace vůči externímu SMTP serveru. Bez toho je komunikace buď nezabezpečená, nebo (pravděpodobněji) ji cílový server odmítne.

Zkrátka a dobře, pro jednoduché a spolehlivé notifikace z Proxmoxu je lepší využít **externí SMTP server**, který se postará o veškerou složitou práci s doručením a reputací. A právě k tomu nám poslouží Gmail.

## Zasílání e-mailů přes SMTP server Googlu (Gmail)

Jak už jsem zmínil, Proxmox bohužel nepodporuje **OAuth** pro přímé přihlašování k Google účtu. To ale neznamená, že jsme bez šance! Google naštěstí nabízí řešení v podobě **hesel aplikací** (App Passwords), které nám umožní se bezpečně autentizovat.

**Předpoklady:**

- Funkční instalace Proxmoxu VE.
- **Google účet** (Gmail), který chcete použít pro odesílání notifikací.

**Krok za krokem:**

### 1. Nastavení dvoufázového ověření (2FA) na Google účtu

Toto je **naprosto klíčový krok**. Hesla aplikací můžete generovat pouze v případě, že máte na svém Google účtu aktivní **dvoufázové ověření (2FA)**. Pokud ho ještě nemáte, je nejvyšší čas si ho nastavit. Zvýšíte tím bezpečnost svého účtu o stovky procent.

1. Přejděte na [https://myaccount.google.com/security](https://myaccount.google.com/security).
2. Najděte sekci "Jak se přihlašujete do Googlu" a klikněte na "**Dvoufázové ověření**".
3. Postupujte podle pokynů k nastavení (nejčastěji se používá ověření přes telefon, aplikaci Google Authenticator nebo hardwarový klíč).

### 2. Vygenerování hesla pro aplikaci

Jakmile máte 2FA aktivní, můžete vygenerovat speciální heslo, které Proxmox použije pro přihlášení k SMTP serveru Googlu.

1. Přejděte na [https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords).
2. Pravděpodobně budete muset znovu potvrdit svou identitu (heslem a 2FA).
3. Do textového pole zadejte výstižný název, např. **Proxmox Notifikace**, a klikněte na "**Vytvořit**".
4. Google vám zobrazí **16místné heslo** v obdélníku (např. `abcd efgh ijkl mnop`). **Toto heslo si ihned zkopírujte a uložte na bezpečné místo!** Po zavření okna ho již nikdy znovu neuvidíte. Pro zadávání do Proxmoxu budete používat heslo bez mezer.

### 3. Nastavení SMTP v Proxmoxu

Teď už jen zbývá říct Proxmoxu, jak má e-maily posílat. Celé nastavení pohodlně provedete přímo ve webovém rozhraní.

1. Přihlaste se do webového rozhraní Proxmoxu.
2. V levém stromovém menu klikněte na nejvyšší úroveň **Datacenter**.
3. V pravé části vyberte záložku **Notifications**.
4. Klikněte na tlačítko "**Add**" a z nabídky zvolte "**SMTP**".

Nyní vyplňte formulář s následujícími údaji:

- **Name:** `gmail-smtp` (nebo jiný vlastní název)
- **Server:** `smtp.gmail.com`
- **Port:** `465` (pro šifrování TLS)
- **Encryption:** Vyberte **TLS**.
- **Username:** Zde zadejte **celou vaši e-mailovou adresu Gmailu** (např. `vas.home.lab@gmail.com`).
- **Password:** Sem vložte **vygenerované 16místné heslo aplikace BEZ MEZER**, které jste si uložili v předchozím kroku.
- **From Address:** Zadejte **stejnou e-mailovou adresu jako Username** (např. `vas.home.lab@gmail.com`). Tato adresa se zobrazí jako odesílatel zpráv.
- **Recipient(s):** vyberte ze selectboxu uživatele, který bude dostávat notifikace.
- **Comment:** Volitelně můžete přidat poznámku.

Po vyplnění všech údajů klikněte na tlačítko "**Add**".

Nyní by měl být Proxmox připravený posílat e-maily. Nejlepší způsob, jak to ověřit, je poslat si testovací zprávu. Klikněte na tlačítko **Test**.

Pokud je vše správně nastaveno, měli byste během několika sekund obdržet testovací e-mail do vaší schránky. Pokud ne, pečlivě zkontrolujte nastavení, zejména správně zkopírované heslo aplikace (opravdu bez mezer!). Také se ujistěte, že máte stále aktivní 2FA. Pro jistotu se podívejte i do složky Spam/Nevyžádaná pošta.

## Bezpečnostní doporučení a dobrá praxe

- **Heslo aplikace je klíčové:** Generujte heslo aplikace pouze tehdy, když ho potřebujete, a nikdy ho s nikým nesdílejte. Pokud máte pocit, že bylo kompromitováno, okamžitě ho zrušte v nastavení Google účtu a vygenerujte nové.
- **Pravidelná kontrola:** Občas si pošlete testovací e-mail, abyste ověřili, že notifikace stále fungují. Google může čas od času měnit své bezpečnostní politiky.
- **Vyhrazený účet:** Zvažte použití speciálního Google účtu pouze pro notifikace z vašeho Home Labu a dalších služeb. Oddělením tohoto účtu od vašeho primárního snížíte potenciální rizika.
- **Notifikace jako doplněk:** E-mailové notifikace jsou skvělé, ale doporučuji je kombinovat i s dalšími formami monitoringu, například s pomocí nástrojů jako **Uptime Kuma, Grafana** nebo **Prometheus**, které vám dají ucelený přehled o stavu vašeho Home Labu v reálném čase.

Máte vlastní zkušenosti s nastavením notifikací v Proxmoxu? Používáte jiného poskytovatele SMTP? Podělte se o své tipy a triky v komentářích!