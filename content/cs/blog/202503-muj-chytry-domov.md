+++
date = '2025-03-22T21:41:02Z'
title = 'Můj chytrý domov: Když technologie spolupracují'
tags = ['ABB free@home', 'HomeKit', 'Home Assistant', 'Matter']
+++

Moje cesta k chytré domácnosti začala nenápadně – LED páskem za televizí. Dnes už propojuji světla, rekuperaci, multimédia i bezpečnostní systémy v jeden „_inteligentní_“ celek. Nabízím Vám pohled na to, jaké technologie u nás doma spolupracují.

## Home Assistant jako mozek celého systému

Home Assistant běží lokálně a slouží jako **centrální integrační platforma**, která spojuje různorodé technologie:

- **ABB free@home**: páteř elektroinstalace a ovládání světel, zásuvek i žaluzií
- **Zigbee (ZHA)** a **Matter**: moderní protokoly pro přímé připojení zařízení
- **Node-RED**: vizuální nástroj pro automatizace
- **Google Cast a Apple TV**: integrace multimédií do scén
- **Modbus TCP**: průmyslový protokol pro řízení rekuperace
- **TrueNAS**: domácí server pro data a zálohy
- **NextDNS a Mikrotik**: kontrola nad sítí a zabezpečením

## Propojení různých světů

### Osvětlení v ABB free@home + Zigbee světla

Pomocí **Emulated Hue** „_předstírám_“, že Zigbee světla jsou Hue kompatibilní, takže je **ABB free@home** dokáže ovládat. Výsledkem je jednotné ovládání všech světel přes nástěnné ovladače.
### Chytrá koupelna s Nest Mini

Díky **Node-RED** a **Home Assistant** mohu z displeje ABB free@home nebo např. Zigbee tlačítkem vyvolat různé scény, spustit rádio.

### Matter a budoucnost kompatibility

**Matter** umožňuje hladké propojení mezi ABB a dalšími výrobci. Využívám propojení s **Apple HomeKit**, díky kterému mohu ovládat zařízení jak z prostředí HomeKit (přes aplikaci Domácnost nebo Siri), tak i z ABB free@home.

### Rekuperace Jablotron přes Modbus TCP

Home Assistant komunikuje přímo s rekuperací. Získávám data o **CO₂, teplotě a vlhkosti** z obývacího pokoje, ložnic, ... a podle toho řídím **větrání a boost režimy** – ručně i automatizovaně.

## Co mě k tomu vedlo?

Chtěl jsem systém, který mi dá **plnou kontrolu**, běží **lokálně**, a zároveň zvládne propojit **různé výrobce**. Home Assistant splnil všechna očekávání – je otevřený, flexibilní a neustále se rozvíjí díky skvělé komunitě.
