+++
date = '2025-04-03T12:08:37Z'
title = 'Co dělat bezprostředně po kybernetickém útoku'
tags = ['kybernetická bezpečnost', 'zálohování', 'reakce na incident']
comments = true
+++

## Praktický krizový scénář pro ICT týmy a vedení organizací

Kybernetický útok může přijít kdykoli. Bez varování, bez rozlišování. Ať už jde o ransomware, cílený útok na infrastrukturu nebo „jen“ únik dat, první hodiny po incidentu rozhodují o všem.

Na této stránce najdete praktický přehled kroků, které je třeba provést ihned po zjištění kybernetického útoku. Vyzkoušeno v praxi, ověřeno i oficiálními doporučeními.

{{< resized-img src="cyber-attack.png" size="800x jpg" alt="Cyber attack" >}}

## 🛑 1. Izolace napadených zařízení a sítí

**Cíl:** zabránit dalšímu šíření útoku.

- Odpojit napadené pracovní stanice a servery ze sítě.
- Odstavit Wi-Fi, VPN, internetovou konektivitu.
- Pozastavit automatické zálohy a synchronizace (např. OneDrive, cloud backupy).
- Odpojit zálohovací server od sítě i elektřiny.

> ⚠️ Nezapomeňte, že ransomware může čekat v síti dny, týdny nebo déle. Rychlá izolace = nižší škody.

## 🔍 2. Zachování důkazů

**Cíl:** umožnit forenzní analýzu a efektivní vyšetřování.

- Neformátovat ani neobnovovat zařízení.
- Uchovat disky, logy, snapshoty VM.
- Zajistit bitové kopie napadených systémů.
- Vytáhnout logy z firewallů, IDS/IPS sond a síťových prvků.

> 💡 Všechny důkazy uložte na bezpečné místo offline.

## 📢 3. Interní informování a aktivace krizového štábu

**Cíl:** koordinovaný postup bez chaosu.

- ICT tým informuje vedení, manažera kybernetické bezpečnosti, DPO, právníka.
- Stanovte osobu odpovědnou za komunikaci – interně i navenek.
- Spusťte krizový plán, pokud jej máte (a měli byste).

## 📋 4. Spuštění plánu reakce na incident

**Cíl:** neztrácet čas vymýšlením, co dál.

- Použijte připravené postupy a checklisty.
- Vyhodnoťte rozsah útoku – napadené systémy, dopad.
- Rozhodněte o nutnosti zapojení externí pomoci (např. CSIRT, specializovaná firma).

> 🛠️ Pokud plán nemáte, je to poslední šance improvizovat systematicky.

## 📞 5. Komunikace mimo kompromitovanou infrastrukturu

**Cíl:** udržet informovanost bez rizika úniku.

- Přepněte komunikaci na alternativní kanály (firemní GSM, osobní e-maily).
- Nikdy neřešte situaci přes napadené systémy (např. interní e-mail).
- Dokumentujte všechna rozhodnutí a opatření.

## ⏳ Obnova? Záleží na dopadu

Podle rozsahu škod může obnova trvat:

- **Malý dopad**: 3 pracovní dny
- **Střední dopad**: 2 týdny
- **Velký dopad**: až 3 měsíce
- **Zásadní dopad**: 9–15 měsíců

> 🧯 Měřítkem není jen počet serverů, ale i funkčnost záloh, dostupnost personálu a hardware.

## 💾 Zálohování: kdo nemá, ten zapláče

Základem přežití je **pravidlo 3–2–1**:

- 3 kopie dat
- 2 různá úložiště
- 1 offline nebo off-site

Důležitá je také **retence záloh** – uchování starších verzí (ideálně "xx" měsíců zpětně), protože útok mohl být v síti dlouho předem.

## 🧠 Poučení pro příště

Z útoku je třeba si odnést:

- Jaké chyby umožnily útočníkovi průnik?
- Co selhalo v reakci?
- Co musíme změnit?

Vhodné je incident **formálně vyhodnotit** (tzv. _post-incident review_), ideálně s externím partnerem.

## 🔚 Závěrem

Kybernetické útoky jsou realitou. Nejde o to, zda se stanou, ale kdy.

Připravte si plán. Trénujte. Zálohujte. A buďte připraveni zachovat chladnou hlavu.
