+++
date = '2024-03-09T13:05:18Z'
title = 'Měření spotřeby vody'
kategorie = ['návod']
tags = ['ESP32', 'AI', 'Home Assistant']
ShowToc = true
TocOpen = true
comments = true
+++

Odečet stavu vodoměru pomocí REST API. Návod pro ty, kteří nevyužívají MQTT.
## 🤖 Ai-on-the-edge-device
Využití AI (Artificial intelligence / umělá inteligence) a zařízení s procesorem ESP32 ke vzdálenému odečtu energií.
[Ai-on-the-edge-device](https://github.com/jomjol/AI-on-the-edge-device)

### ⭐ Přednosti řešení
1. Malé a levné zařízení;
2. Kamera včetně LED diody pro přísvit;
3. Webová administrace pro nastavení odečtu;
4. Podpora pro Home Assistant, InfluxDB, MQTT, REST API;
5. Nízká spotřeba zařízení ESP32;

### 📸 Dokončení nastavení zařízení ESP32-CAM
Aby vše správně fungovalo a zobrazily se všechny údaje při volání REST API, nesmí být zařízení v chybovém stavu.

Může se stát, že bude chybně zobrazena informace v části `Previous Value`. Je to dáno tím, že se nám odečetla chybně hodnota (nejspíše před dokončením úvodního nastavení). Zařízení hlásí chybu (Rate too high ...) a je nutné to opravit.

```
Settings -> Set "Previous Value" -> Enter new "previous value"
```
Nastavte zde stejnou hodnotu, jako je v části `Current "previous value"`.

## 🔗 Integrace s HA pomocí REST API
Nevyužívám pro připojení k ESP32-CAM MQTT, ale REST API. Home Assistant v pravidelných (5 minut) intervalech volá URL adresu a jsou mu vrácena data ve formátu JSON.

```
http://<IP.adresa.ESP32.cam>/json
```

Získaná data vypadají takto:
```

{
    "main": {
        "value": "393.6500",
        "raw": "393.6500",
        "pre": "393.6500",
        "error": "no error",
        "rate": "0.000000",
        "timestamp": "2024-03-09T20:45:13+0100"
    }
}
```

Do konfiguračního souboru configuration.yaml (nebo dle Vašeho nastavení) je potřeba přidat senzor a šablonu:
```
sensor:
- platform: rest
  name: "Technická místnost vodoměr json"
  unique_id: technicka_mistnost_vodomer_json
  resource: http://<IP.adresa.ESP32.cam>/json
  json_attributes:
    - main
  value_template: >
    {% if value_json.value | length > 0 %}
        {{ value_json.value }}
    {% else %}
        {{ value_json.pre }}
    {% endif %}
  headers:
    Content-Type: application/json
  scan_interval: 300

template:
  sensor:
  - name: "Technická místnost vodoměr"
    unique_id: technicka_mistnost_vodomer
    state: "{ state_attr('sensor.technicka_mistnost_vodomer_json','main')['value']|float }}"
    unit_of_measurement: 'm³'
    device_class: water
    state_class: total_increasing
    icon: mdi:gauge
```

Nyní stačí zkontrolovat, zda je vše správně zapsáno v konfiguračním souboru (Nástroje pro vývojáře -> Zkontrolovat nastavení) a restartovat Home Assistanta. Po restartu vidíme 2 nové entity a můžeme přidat nový senzor do panelu "Energie".

[![Open your Home Assistant instance and show your dashboard configs.](https://my.home-assistant.io/badges/lovelace_dashboards.svg)](https://my.home-assistant.io/redirect/lovelace_dashboards/)

Otevřeme si panel Energie a do části Spotřeba vody přidáme náš nový senzor. Následně uvidíme v dashboardu spotřebu vody.

{{< resized-img src="mereni-spotreby-vody.png" size="800x jpg" alt="Spotřeba vody" >}}
