+++
date = '2025-05-30T17:05:43Z'
draft = false
title = 'Proxmox Cluster: 2 nody a QDevice'
kategorie = ['návod']
tags = ['Proxmox', 'cluster', 'qdevice', 'HA']
ShowToc = true
TocOpen = true
comments = true

+++

Provozování robustního a spolehlivého homelabu je cílem všech technologických nadšenců. Potřebuji virtualizaci a zajištění vysoké dostupnosti (HA) pro důležité služby, jako je například Home Assistant. Proxmox VE je skvělou platformou pro virtualizaci a navíc přímo umožňuje vytvořit cluster. To otevírá dveře k pokročilým možnostem, jako je migrace virtuálních strojů (VM) a kontejnerů (LXC) a právě vysoká dostupnost.

Proxmox cluster vyžaduje pro správné fungování (quorum) minimálně tři nody. Co ale dělat, když máte k dispozici pouze dva fyzické servery? Řešením je tzv. QDevice, externí arbitr, který pomůže clusteru dosáhnout kvóra. V tomto článku si ukážeme, jak vytvořit cluster se dvěma nody a jako QDevice využít virtuální stroj běžící na TrueNAS.

## Proč cluster a proč QDevice?

### Výhody Proxmox clusteru

- **Live migrace VM:** Možnost přesouvat běžící virtuální stroje mezi nody bez výpadku. To je skvělé i při údržbě hardwaru. Je nutné počítat s tím, že se kopíruje obsah operační paměti (RAM) VM, což může u strojů s velkou RAM trvat déle. Pro některé scénáře a konfigurace může být rychlejší VM kontrolovaně vypnout a zapnout na cílovém nodu.
- **Migrace LXC kontejnerů:** Proxmox za určitých podmínek podporuje online migraci LXC (se sdíleným úložištěm), v praxi může být pro zajištění konzistence nutný krátký restart kontejneru na cílovém nodu, případně se pro jednodušší scénáře bez sdíleného úložiště využívá offline migrace (vypnutí, přesun, zapnutí).
- **Vysoká dostupnost (HA):** Pokud jeden z nodů selže, Proxmox HA automaticky restartuje definované VM/LXC na druhém, funkčním nodu. Ideální pro služby jako Home Assistant, které chci mít neustále online.
- **Centralizovaná správa:** Spravujete všechny nody z jednoho webového rozhraní.

### Problém se dvěma nody

Kvórum je mechanismus, který zajišťuje, že cluster může učinit rozhodnutí (např. o tom, který node je "master" nebo zda spustit HA procesy) pouze tehdy, když je online většina nodů. Tím se předchází situaci zvané *split-brain*, kdy by obě části rozděleného clusteru mohly fungovat nezávisle a způsobit nekonzistenci dat.

Se dvěma nody nemůže cluster automaticky určit, která polovina je ta *správná* v případě ztráty komunikace mezi nimi. Zde přichází na řadu **QDevice (Quorum Device)**. Je to speciální proces (corosync-qnetd), který běží na třetím, nezávislém stroji a funguje jako další hlas při rozhodování. Pro cluster se dvěma nody a jedním QDevice je kvórum dosaženo, pokud jsou online alespoň dva ze tří *hlasujících* (tedy oba nody, nebo jeden node a QDevice).

**Co potřebujeme:**

1. **Dva Proxmox VE servery:** Oba nody musí být aktuální (`proxmox1` a `proxmox2`).
2. **RPi** nebo jiný **linuxový server (já mám TrueNAS):** Na tomto serveru vytvoříme malý virtuální stroj (používám Debianem), kde poběží `corosync-qnetd` (nazval jsem jej `qdevice-vm`).
3. **Síťová konektivita:**
    - Všechny tři stroje (proxmox1, proxmox2, qdevice-vm) musí být ve stejné síti a musí na sebe vidět.
    - Doporučuje se mít pro clusterovou komunikaci dedikovanou síťovou kartu/VLAN, ale pro menší homelaby to není striktně nutné (může však ovlivnit výkon a stabilitu).
    - Jsou doporučeny statické IP adresy pro všechny stroje.
4. **Synchronizovaný čas:** Všechny nody v clusteru (včetně QDevice) musí mít nastaveny NTP klienty. Rozdíly v čase mohou způsobit vážné problémy s clusterem.
5. **Vždy zálohujte!**

## Krok 1: Příprava Proxmox nodů

Na obou Proxmox nodech (`proxmox1` i `proxmox2`):

1. **Aktualizujte systém:** např. přes GUI nebo CLI.
2. **Zkontrolujte `/etc/hosts`:** Ujistěte se, že každý nod zná IP adresu a hostname ostatních nodů:
    ```
    127.0.0.1 localhost
    <IP_ADRESA_PROXMOX1> proxmox1.vasadomena.local proxmox1
    <IP_ADRESA_PROXMOX2> proxmox2.vasadomena.local proxmox2
    <IP_ADRESA_QDEVICE_VM> qdevice-vm.vasadomena.local qdevice-vm
    ```
3. **Nastavte NTP klienta na Proxmoxu:** V novějších verzích Proxmox VE (konkrétně od verze 7) je jako výchozí NTP démon používán **`chrony`**. Přihlaste se do GUI, v levém navigačním panelu klikněte na **název vašeho nodu** (serveru Proxmox). Zvolte `System - Time`. Vše by mělo být předkonfigurováno.
4. **Nastavte NTP na Debianu:** Já jsem musel doinstalovat:
    ```
    apt install systemd-timesyncd -y
    ```
5. **Konfigurace NTP serverů (Debian):** Upravte soubor `/etc/systemd/timesyncd.conf`. Odkomentujte nebo přidejte řádek `NTP`
	```
	[Time]
	NTP=0.cz.pool.ntp.org 1.cz.pool.ntp.org
	```
    Restartujte službu: `systemctl restart systemd-timesyncd` a zkontrolujte stav: `timedatectl status`.

## Krok 2: Vytvoření clusteru (`proxmox1`)

Vytvořit nový cluster je možné pohodlně pomocí grafického GUI: `Datacenter - Cluster - Create Cluster`. Nebo přes SSH na `proxmox1` a spusťte příkaz pro vytvoření nového clusteru:

```
pvecm create NAZEV_VASEHO_CLUSTERU
```

## Krok 3: Přidání druhého nodu (`proxmox2`)

Přejděte na node2 (proxmox2) do webového rozhraní (GUI), opět jděte do nastavení clusteru a zvolte tlačítko `Join Cluster`. Potřebné informace získáte z `proxmox1`.

Jako alternativní řešení je možné použít SSH na `proxmox2`, např:

```
pvecm add 192.168.1.10 # Nahraďte IP adresou vašeho proxmox1
```

Budete vyzváni k zadání root hesla pro `proxmox1` a potvrzení otisku SSH klíče. Po úspěšném přidání se `proxmox2` připojí k clusteru. To může chvíli trvat.

**Důležité:** Po přidání nodu do clusteru se veškerá konfigurace spravuje centrálně. Lokální konfigurační soubory (např. `/etc/pve/storage.cfg`) na jednotlivých nodech by se již neměly ručně upravovat.

## Krok 4: Quorum

V tuto chvíli máte cluster se dvěma nody. Zkuste si ve webovém rozhraní Proxmoxu (na kterémkoli z nodů) zobrazit stav clusteru. Pravděpodobně uvidíte varování.

```
pvecm status
```

Uvidíte něco jako:

```
...
Votequorum information
----------------------
Expected votes:   3  <-- Očekává 3 hlasy pro kvorum (2 nody + QDevice)
Highest expected: 3
Total votes:      2  <-- Aktuálně má jen 2 hlasy (2 nody)
Quorum:           2
Flags:            TwoNode TieBreaker
...
Membership information
----------------------
    Nodeid      Votes Name
0x00000001          1 proxmox1 (local)
0x00000002          1 proxmox2
```

Dokud nepřidáme QDevice, cluster nebude mít stabilní kvórum, pokud jeden z nodů selže.

## Krok 5: Nastavení Corosync QNet Daemon (QDevice) na TrueNAS VM

Jak bylo zmíněno, QDevice poběží ve virtuálním stroji na TrueNAS.

**Vytvořte VM na TrueNAS:**
- Vytvořte si malý VM. Postačí 1 vCPU, 512MB-1GB RAM a malý disk (cca 8-10GB).
- Nainstalujte oblíbenou distribuci (používám Debian).
- Nastavte statickou IP adresu pro tento VM (např. `192.168.1.12`) a ujistěte se, že má funkční DNS a NTP.

**Instalace `corosync-qnetd` na `qdevice-vm`:** Přihlaste se na `qdevice-vm` přes SSH a nainstalujte balíček `corosync-qnetd`:
    ```
    sudo apt update
    sudo apt install corosync-qnetd -y
    ```

**Konfigurace `corosync-qnetd`:** Výchozí konfigurace by měla být pro většinu případů dostatečná. Služba by se měla automaticky spustit a naslouchat na portu `5403/TCP`. Ověřte, že služba běží:
    
    ```
    systemctl status corosync-qnetd
    ```

Pokud máte na `qdevice-vm` aktivní firewall (např. `ufw`), ujistěte se, že povolujete příchozí spojení na TCP port 5403 z IP adres vašich Proxmox nodů.

## Krok 6: Přidání QDevice do Proxmox Clusteru

Vraťte se na jeden z Proxmox nodů (např. `proxmox1`).

Nejprve se ujistíme, že na svém Proxmox nodu (odkud se budeme připojovat na QDevice VM) máme vygenerovaný SSH klíč. Já jsem jej měl již vygenerovaný, má obvykle název `id_rsa.pub` a je uložen v adresáři `~/.ssh/`. Je možné jej vygenerovat např. `ssh-keygen -t rsa -b 4096`.

Obsah souboru z `.ssh/id_rsa.pub` jsem zkopíroval do qdevice-vm na konec souboru `/root/.ssh/authorized_keys`. SSH server v Debian nemá povoleno přihlášení uživatele root pomocí hesla, proto nebude možné použít: `ssh-copy-id root@qdevice-vm_ip_adresa`

Přidejte QDevice do clusteru:

```
pvecm qdevice setup 192.168.1.12 # IP adresa qdevice-vm
```

Po přidání QDevice chvíli počkejte a poté znovu zkontrolujte stav clusteru na libovolném Proxmox nodu:

```
pvecm status
```

Nyní byste měli vidět, že cluster očekává 3 hlasy a má 3 hlasy (pokud jsou oba nody a QDevice online a komunikují):

```
...
Votequorum information
----------------------
Expected votes:   3
Highest expected: 3
Total votes:      3  <-- Nyní máme 3 hlasy!
Quorum:           2
Flags:            Quorate Qdevice
...
Membership information
----------------------
    Nodeid      Votes Name
0x00000001          1 proxmox1 (local)
0x00000002          1 proxmox2
0x00000000          1 QDevice (votes 1) <-- Zde je náš QDevice!
```

Pokud vidíte `Quorate` a `Qdevice` ve Flags a `Total votes: 3`, je vše v pořádku. Váš cluster má nyní stabilní kvórum i se dvěma fyzickými nody.

## Krok 7: Konfigurace vysoké dostupnosti (HA) pro Home Assistanta (nebo jiné VM/LXC)

Nyní, když máme funkční cluster, můžete začít využívat vysokou dostupnost.

1. **Přejděte do webového rozhraní Proxmoxu.**
2. **Vyberte Datacenter -> HA.**
3. **Klikněte na "Add".**
4. **Vyberte VM nebo LXC**, pro které chcete HA aktivovat (např. váš Home Assistant VM).
5. **Nastavte parametry:**
    - **Max Restart:** Kolikrát se má Proxmox pokusit restartovat službu na stejném nodu, než ji označí za selhanou.
    - **Max Relocate:** Kolikrát se má Proxmox pokusit přemístit (migrovat) službu na jiný nod.
    - **Group (volitelné):** Můžete definovat skupiny pro HA zdroje.

Proxmox HA se nyní postará o váš Home Assistant. Pokud node selže, HA se pokusí VM restartovat na druhém dostupném nodu (za předpokladu, že druhý node a QDevice jsou online a udržují kvórum).

## Důležité pro HA

- **Sdílené úložiště (ideální stav):** Pro live migraci a HA je ideální mít sdílené úložiště (např. NFS, iSCSI, Ceph) dostupné pro oba Proxmox nody. Disky VM/LXC jsou pak na jednom místě a nemusí se kopírovat.
- **Replikace (aktuální řešení):** Proxmox nabízí funkci replikace. Ta umožňuje pravidelně synchronizovat disky VM a LXC kontejnerů z jednoho nodu na druhý. V případě selhání nodu nebo při plánované migraci jsou tak data již přítomna na cílovém nodu, což výrazně zrychluje proces přesunu/obnovy oproti kopírování celého disku v reálném čase.
- **Testování:** Důkladně otestujte HA konfiguraci simulací výpadku jednoho z nodů (např. jeho vypnutím), abyste se ujistili, že vše funguje podle očekávání, včetně rychlosti obnovy s využitím replikovaných dat.