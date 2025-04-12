+++
date = '2025-04-11T19:20:41Z'
draft = false
title = 'Jak jsem nakonfiguroval Mikrotik pro bezpečnou a přehlednou síť'
kategorie = ['návod']
tags = ['Mikrotik', 'DNS', 'VLAN', 'firewall']
ShowToc = true
TocOpen = true
comments = true

+++

Konfigurace Mikrotiku může být robustní a bezpečná, pokud se držíte několika důležitých principů. Tento článek shrnuje mé osvědčené postupy a konkrétní příklady, jak na to, aby vaše síť byla přehledná, segmentovaná a chráněná.

V dnešní době už domácí síť dávno není jen o připojení počítače k internetu. Stala se digitálním srdcem našich domovů, propojujícím notebooky, telefony, tablety, chytré televize, herní konzole, ale i stále rostoucí armádu IoT (Internet of Things) zařízení – od chytrých termostatů a osvětlení, přes bezpečnostní kamery a senzory, až po chytré ledničky nebo pračky. S tímto nárůstem komplexnosti a počtu připojených zařízení ale exponenciálně roste i důležitost zabezpečení a přehlednosti. Mikrotik nám dává nástroje, jak tuto komplexnost zvládnout.

## ⚠️ Důležité upozornění

> Následující konfigurační příklady slouží jako ilustrace principů a přístupu ke konfiguraci. Nejsou určeny pro přímé kopírování (Copy/Paste) do vašeho zařízení bez pochopení jednotlivých kroků a jejich přizpůsobení vaší konkrétní situaci a síťové topologii. Každá síť je unikátní!

> Konfigurace je navíc demonstrována s ohledem na **Mikrotik RB3011**, který využívá specifický switch čip (QCA8337). **Konfigurace VLAN na úrovni bridge portů (`/interface bridge port` – nastavení PVID, frame-types, ingress-filtering atd.) se může lišit u jiných modelů Mikrotik routerů** s odlišnými switch čipy. Vždy si prostudujte dokumentaci a příklady pro váš konkrétní model hardwaru, zejména pokud jde o detailní nastavení VLAN tagování na fyzických portech. Tento článek se zaměřuje spíše na konfiguraci na úrovni RouterOS (bridge, VLAN rozhraní, IP, DHCP, Firewall, VPN), která je obecnější.

## 1. Klíčové vlastnosti konfigurace

### ✅ Pravidelné aktualizace

RouterOS je aktivně vyvíjen a jsou pravidelně vydávány aktualizace, které nejen přidávají nové funkce, ale především opravují bezpečnostní zranitelnosti. Udržovat systém aktuální je základním kamenem bezpečnosti. Dostupnost nových verzí snadno ověříte a nainstalujete:

```
/system package update
check-for-updates
# Pokud je dostupná nová verze:
download
install
# Router se restartuje a nainstaluje aktualizace
```

### Interface a bridge: proč a jak

Bridge (most) umožňuje spojit více fyzických nebo virtuálních rozhraní do jedné logické L2 sítě. Na tento bridge pak můžete aplikovat jednotná nastavení IP adresace, DHCP, firewallu a především VLAN. Pro moderní konfiguraci VLAN je klíčové zapnout **VLAN filtering**.

_Nejprve je dobré přejmenovat fyzická rozhraní pro lepší orientaci:_

```
/interface ethernet
set [find default-name=ether2] name=ether2-LAN
set [find default-name=ether3] name=ether3-LAN
# ... a tak dále pro další porty určené pro LAN
```

_Vytvoření bridge pro LAN:_

> **Poznámka k aktivaci `vlan-filtering=yes`:** Je osvědčenou praxí nastavit `vlan-filtering=yes` až poté, co máte připravenou základní konfiguraci VLAN (VLAN rozhraní na bridge, jejich IP adresy, DHCP) a především jste si **jisti funkčním přístupem pro správu routeru** (správně nastavený PVID a VLAN na management portu v `/interface bridge port` a `/interface bridge vlan`). Předčasná aktivace bez korektního nastavení může snadno vést ke ztrátě spojení s routerem. Finální ladění a ověřování správné funkce VLAN tagování a pravidel pak probíhá až s aktivním filtrováním.

```
/interface bridge
# Vytvoříme bridge
add name=bridge-LAN vlan-filtering=no protocol-mode=none comment="LAN bridge"
```

_Přidání fyzických portů do bridge:_

```
/interface bridge port
# Porty přidané do bridge budou nyní součástí L2 segmentu bridge-LAN
add bridge=bridge-LAN interface=ether2-LAN
add bridge=bridge-LAN interface=ether3-LAN
# Poznámka: Detailní nastavení VLAN tagování (PVID, povolené VLAN) na těchto portech
# závisí na switch čipu vašeho modelu a přesahuje rámec tohoto úvodu.
# Prostudujte wiki.mikrotik.com pro váš model a "Bridge VLAN Table".
```

### Podpora VLAN (802.1q)

Virtuální LAN (VLAN) umožňuje logicky rozdělit jednu fyzickou síť na více oddělených segmentů. VLAN rozhraní vytváříme přímo na tomto bridge.

```
/interface vlan
# Vytvoříme virtuální rozhraní pro každou VLAN na našem hlavním bridge
add interface=bridge-LAN name=vlan10-doma vlan-id=10 comment="VLAN pro důvěryhodná zařízení"
add interface=bridge-LAN name=vlan20-iot vlan-id=20 comment="VLAN pro IoT zařízení"
add interface=bridge-LAN name=vlan30-guest vlan-id=30 comment="VLAN pro hosty"
# ... případně další VLAN dle potřeby (kamery, management)
```

### IP adresace a DHCP Server

Každá VLAN musí mít vlastní IP rozsah a bránu. Tu nastavíme přímo na VLAN rozhraní Mikrotiku. DHCP server pak bude přidělovat adresy zařízením v dané VLAN.

_Nastavení IP adresy pro router na každé VLAN (brána pro klienty):_

```
/ip address
add address=192.168.10.1/24 interface=vlan10-doma network=192.168.10.0 comment="Brána pro VLAN doma"
add address=192.168.20.1/24 interface=vlan20-iot network=192.168.20.0 comment="Brána pro VLAN IoT"
add address=192.168.30.1/24 interface=vlan30-guest network=192.168.30.0 comment="Brána pro VLAN hosté"
# ... a tak dále pro další VLAN
```

_Nastavení DHCP Poolu (rozsahu adres k přidělení):_

```
/ip pool
add name=pool_vlan10 ranges=192.168.10.2-192.168.10.199 comment="Rozsah pro VLAN doma"
add name=pool_vlan20 ranges=192.168.20.2-192.168.20.199 comment="Rozsah pro VLAN IoT"
add name=pool_vlan30 ranges=192.168.30.2-192.168.30.199 comment="Rozsah pro VLAN hosté"
# ...
```

_Konfigurace DHCP serveru pro každou VLAN:_

```
/ip dhcp-server
add name=dhcp_vlan10 interface=vlan10-doma address-pool=pool_vlan10 lease-time=1h disabled=no comment="DHCP pro VLAN doma"
add name=dhcp_vlan20 interface=vlan20-iot address-pool=pool_vlan20 lease-time=1h disabled=no comment="DHCP pro VLAN IoT"
add name=dhcp_vlan30 interface=vlan30-guest address-pool=pool_vlan30 lease-time=1h disabled=no comment="DHCP pro VLAN hosté"
# ...
```

_Nastavení síťových parametrů pro DHCP (brána, DNS):_

Fragment kódu

```
/ip dhcp-server network
# Pro každou síť definujeme bránu (IP adresa routeru v dané VLAN) a DNS server
# Zde jako DNS server používáme IP adresu routeru, který bude dotazy řešit (viz dále)
add address=192.168.10.0/24 gateway=192.168.10.1 dns-server=192.168.10.1 domain=doma.local comment="Parametry sítě pro VLAN doma"
add address=192.168.20.0/24 gateway=192.168.20.1 dns-server=192.168.20.1 domain=iot.local comment="Parametry sítě pro VLAN IoT"
add address=192.168.30.0/24 gateway=192.168.30.1 dns-server=192.168.30.1 domain=guest.local comment="Parametry sítě pro VLAN hosté"
# ...
```

### Lokální DNS server a cache

Mikrotik může fungovat jako DNS resolver a cache pro vaši lokální síť. Zařízení v síti se pak ptají Mikrotiku, který dotazy buď zodpoví z cache, nebo je přepošle na nakonfigurované upstream servery.

```
/ip dns
# Povolíme Mikrotiku odpovídat na DNS dotazy z lokální sítě
set allow-remote-requests=yes cache-size=4096KiB
# Upstream servery nastavíme v sekci DoH nebo pomocí 'set servers=...'
```

#### Bezpečné DNS: Strážce internetových adres

Filtrování a šifrování DNS je klíčové pro bezpečnost a soukromí. Můžete využít cloudové služby jako NextDNS (mé oblíbené řešení) nebo Cloudflare/Quad9, případně lokální Pi-hole.

- **Nastavení DNS-over-HTTPS (DoH) s NextDNS (Doporučeno, RouterOS v7+):** Toto zajistí šifrované a filtrované DNS dotazy přímo z vašeho routeru.
    
    Fragment kódu
    
    ```
    /ip dns
    # Nahraďte YOUR_PROFILE_ID vaším unikátním ID z NextDNS
    set use-doh-server="https://dns.nextdns.io/YOUR_PROFILE_ID" verify-doh-cert=yes
    # Můžete smazat nebo zakomentovat klasické DNS servery, pokud používáte DoH
    # set servers=""
    ```
    
- **Vynucení DNS přes Mikrotik:** Jak bylo zmíněno dříve, je **zásadní** nastavit firewallová pravidla (`/ip firewall nat`, `action=redirect` nebo `dst-nat`), která přesměrují veškerý odchozí DNS provoz (port 53 UDP/TCP) z vaší LAN na samotný Mikrotik router. Tím zajistíte, že i zařízení s "natvrdo" nastavenými DNS servery budou procházet přes vaše bezpečné filtrování.

## 2. Segmentace sítě pomocí VLAN

VLAN (Virtual Local Area Network) je technologie, která umožňuje rozdělit fyzickou síť na více logicky oddělených sítí. Je to jako postavit neviditelné zdi uvnitř vaší sítě.

### Proč segmentovat?

- ⚠️ **Bezpečnost:** Zásadní. Pokud dojde k napadení zařízení v jedné VLAN (typicky méně důvěryhodná IoT síť), útočník se nedostane snadno k vašim citlivým datům v jiné VLAN (počítače, NAS). Omezuje laterální pohyb útočníka.
- 📊 **Přehlednost a správa:** Snazší správa pravidel a monitoring provozu pro jednotlivé segmenty.
- ⚖️ **Výkon:** Omezení zbytečného broadcastového provozu na menší segmenty.

### Typické VLANy v domácí síti:

| **VLAN ID** | **Popis**              | **Příklad IP rozsahu** | **Poznámka**                                 |
| ----------- | ---------------------- | ---------------------- | -------------------------------------------- |
| 10          | Důvěryhodná síť (DOMA) | 192.168.10.0/24        | PC, notebooky, telefony, NAS (správa)        |
| 20          | IoT zařízení           | 192.168.20.0/24        | Chytré žárovky, termostaty, senzory, TV      |
| 30          | Hosté (GUEST)          | 192.168.30.0/24        | Návštěvy - přístup jen k internetu           |
| 40          | Kamerová síť (CAMS)    | 192.168.40.0/24        | IP kamery (často bez přístupu k internetu)   |
| 50          | Management (MGMT)      | 192.168.50.0/24        | Přístup ke správě síťových prvků (volitelné) |

### Co potřebujete za zařízení:

Výčet není rozhodně konečný, ale je důležité si uvědomit, že podpora VLAN musí být i u dalších aktivních prvků Vaší domácí sítě.

- Mikrotik router (nebo jiný s podporou VLAN a firewallu).
- Spravovatelné (managed) switche s podporou standardu 802.1q VLAN (např. Mikrotik, TP-Link řady EasySmart/Smart/JetStream, UniFi, Cisco, ...).
- Wi-Fi Access Pointy (AP) s podporou VLAN, pokud chcete mít různé Wi-Fi sítě pro různé VLAN.

## 3. Ochrana VLAN a routeru pravidly firewallu (`/ip firewall filter`)

Firewall je mozkem zabezpečení vaší sítě. Rozhoduje o tom, co smí a nesmí komunikovat. Základní filozofie by měla být **"povolit jen explicitně definovaný a nezbytný provoz, vše ostatní zakázat"**.

**Klíčové principy a osvědčené postupy:**

1. **Používejte Address Lists (`/ip firewall address-list`)**:
    
    - **Proč?** Dramaticky zvyšují přehlednost a zjednodušují správu. Místo konkrétních IP adres pracujete s pojmenovanými skupinami (např. `Trusted_Devices`, `NAS_Servers`, `IoT_Printers`). Změna IP adresy pak znamená úpravu pouze v seznamu, nikoli v mnoha pravidlech.
    - **Příklad:**
        ```
        /ip firewall address-list
        add list=allow_management address=192.168.10.10 comment="Můj PC pro správu"
        add list=servers_NAS address=192.168.10.100 comment="Hlavní NAS"
        add list=printers_IoT address=192.168.20.50 comment="Síťová tiskárna v IoT"
        ```
        
2. **Zabezpečte `input` chain (Ochrana routeru):**
    
    - Řídí provoz směřující na samotný router.
    - **Zásady:** Povolit navázaná spojení, zahodit neplatná, povolit přístup ke správě (Winbox, SSH, WebFig) **pouze** z důvěryhodných IP adres (definovaných v Address List!), povolit lokální DNS/NTP dotazy, povolit ICMP (ping) z LAN, povolit příchozí VPN spojení, **vše ostatní z externích sítí zahodit**.
3. **Konfigurujte `forward` chain (Řízení provozu mezi sítěmi):**
    
    - Řídí provoz procházející routerem (mezi VLANami, do internetu).
    - **Zásady a pořadí pravidel:**
        - **Optimalizace:** Případně povolit FastTrack pro established/related spojení (zvyšuje výkon).
        - **Základ:** Vždy povolit `connection-state=established,related` (umožní odpovědi na již navázaná spojení).
        - **Bezpečnost:** Vždy zahodit `connection-state=invalid`. Zahodit nové pokusy o spojení z WAN, které nejsou součástí DST-NAT pravidla (zabraňuje útokům zvenčí).
        - **Inter-VLAN Pravidla (Jádro segmentace):** Explicitně povolit **pouze nezbytnou** komunikaci mezi VLANami. Používejte Address Listy pro zdroje a cíle a specifikujte porty a protokoly. _Příklad: Povolit `src-address-list=Trusted_Devices` přístup na `dst-address-list=printers_IoT` na port TCP 9100._
        - **Pravidla pro Internet (Egress Filtering):** Povolit přístup z jednotlivých VLAN do internetu (`out-interface-list=WAN`). Pro méně důvěryhodné VLAN (jako IoT) buďte **velmi specifičtí** a povolte jen **nezbytné porty a protokoly**! _Příklad pro IoT: Povolit TCP porty 80, 443, 8883 (Web, MQTT) a UDP porty 123, 53 (NTP, DNS - pokud není přesměrováno)._ VLAN pro hosty by měla mít přístup jen do WAN.
        - **Default Drop Pravidla:** Na konci `forward` chainu umístěte pravidla, která **zahodí veškerý provoz**, který nebyl explicitně povolen předchozími pravidly. Minimálně: zahodit provoz mezi VLANami (`in-interface-list=LAN out-interface-list=LAN`) a ideálně i veškerý zbývající forward provoz.
    - **Logování:** Pro ladění používejte `log=yes` a `log-prefix="..."` na drop pravidlech, abyste viděli, co je blokováno. V produkci logujte uvážlivě.

## 4. VPN: Vzdálený přístup bezpečně s WireGuard

Pro bezpečný přístup do domácí sítě odkudkoli je ideální moderní VPN protokol WireGuard, který Mikrotik skvěle podporuje. Je rychlý a relativně snadno konfigurovatelný.

- **Konfigurace rozhraní WireGuard na Mikrotiku (serveru):**
    
   ```
    /interface wireguard
    add name=wg-home listen-port=50910 mtu=1420 comment="Domácí VPN"
    # Veřejný klíč serveru si zobrazte pomocí /interface wireguard print detail
    ```
    
- **Přidání klienta (peer):** Pro každé klientské zařízení (telefon, notebook) přidáme "peer".
    
    ```
    /interface wireguard peers
    # public-key zkopírujte z konfigurace klienta
    # allowed-address je IP adresa klienta v rámci VPN tunelu
    add interface=wg-home public-key="<veřejný_klíč_klienta_1>" allowed-address=192.168.99.2/32 comment="Mobil"
    ```
    
- **IP adresace pro VPN tunel:** Nastavíme IP adresu pro Mikrotik router _uvnitř_ VPN tunelu.
    
    ```
    /ip address
    add address=192.168.99.1/24 interface=wg-home network=192.168.99.0 comment="WireGuard VPN Gateway IP"
    ```
    
- **Povolení provozu ve firewallu:** Musíme povolit příchozí spojení na WireGuard port (`input` chain) a definovat, kam se mohou připojení klienti dostat (`forward` chain).
    
    ```
    # V INPUT chain: Povolit UDP port pro WireGuard
    /ip firewall filter add chain=input action=accept protocol=udp dst-port=50910 comment="Allow WireGuard UDP"
    
    # Ve FORWARD chain: Povolit provoz z VPN do vybraných interních sítí
    # Ideálně použijte Address List (např. InternalAccessFromVPN) pro cílové sítě
    /ip firewall filter add chain=forward action=accept in-interface=wg-home dst-address-list=InternalAccessFromVPN comment="Allow VPN to Internal Networks"
    
    # Případně NAT pro přístup VPN klientů do internetu
    /ip firewall nat add action=masquerade chain=srcnat out-interface-list=WAN src-address=192.168.99.0/24 comment="Masquerade VPN clients"
    ```

## Závěr

Díky správnému využití bridge s VLAN filtrováním, pečlivé segmentaci pomocí VLAN, promyšlenému nastavení firewallu s důrazem na Address Listy a zabezpečení DNS můžete i v domácích podmínkách provozovat velmi robustní a bezpečnou síť. Mikrotik RB3011 (a další modely) k tomu dává do rukou mocné nástroje, které u běžných spotřebitelských routerů často nenajdete. Pamatujte, že klíčem je pochopení principů a pečlivá konfigurace přizpůsobená vašim potřebám.
