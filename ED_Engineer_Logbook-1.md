# ED Engineer Logbook – IHCO Guide
> Übergabedokument für nachfolgende Claude-Sessions  
> *"Don't Panic."* – Douglas Adams

---

## 🎯 Produktvision
> Ein Tool für die Unterstützung von BGS- und Kolonisierungs-Aktivitäten einer einzelnen Squad – mit Einblick in das weitere Umfeld.

---

## 🌍 Internationalisierung (i18n)
- Die IHCO vereint Spieler aus aller Welt
- **Standardsprache:** Englisch
- Weitere Sprachen nach Bedarf der Squad
- Alle UI-Texte müssen externalisiert werden – keine hardcodierten Strings im Code
- Spracherkennung via `navigator.language`, Fallback immer Englisch
- Geplante Struktur:
```
ihco-guide/
└── locales/
    ├── en.json    # Englisch (Standard)
    ├── de.json    # Deutsch
    └── ...
```

---

## 📍 Aktueller Stand

**Datum:** 2026-05-27  
**Repo:** https://github.com/vonstackelberg-hell/ihco-guide  
**Live:** https://vonstackelberg-hell.github.io/ihco-guide/  

### ✅ Fertiggestellt
- 3D-Sternenkarte mit Three.js (Glow-Partikel, AdditiveBlending)
- GitHub Pages Hosting
- HTML/CSS/JS sauber getrennt
- `embed_map.py` baut portable `index.html` aus Quelldateien (inkl. `ihco_config.js`, `stars.js`, `grid.js`)
- `ihco_filter.py` mit Delta-Merge & Verloren-Logik
- Zwei GitHub Actions Workflows:
  - `update.yml` – Dienstags 06:00 UTC, lädt Spansh-Daten, merged, committet ✅ live getestet
  - `deploy.yml` – Bei jedem Push, baut und deployed GitHub Pages
- `.gitignore` schützt `data/` und `*.gz`
- `README.md` im Repo
- `ihco_config.js` – zentrale Konfigurationsdatei (Zoom, Grid, Hover, Labels)
- OOP-Refactoring: `stars.js` (`BackgroundStars`, `SystemLayer`), `grid.js` (`InfiniteGrid`)
- Infinite Grid mit anti-aliased Shader + Koordinaten-Labels + Toggle
- `sizeAttenuation: false` für alle Sterne – schärfer beim Zoom
- Hover-Threshold dynamisch (`spherical.radius * factor`)
- Raycaster ignoriert unsichtbare Gruppen (Toggle-Bug behoben)
- SVG-Overlay für Verbindungslinien von Nachbarnamen zu Sternen
- `#hover`-Div: gehoverten Stern + Nachbarliste mit Linien zu den Sternen
- Linien werden beim Zoomen neu berechnet

### 🔧 Bekannte Offene Punkte
- `hoverThresholdFactor` noch nicht in Config, aktuell hardcodiert
- `labelMaxNeighbors` in Config ergänzen
- `labelNeighborRadius` ggf. noch tunen
- Duplikate im Repo prüfen: `py/ihco_systems.json`, `data/ihco_systems.json`
- `ihco_map_embedded.html` ggf. noch im Repo (veraltet, löschen)
- UI-Texte noch hardcodiert (Englisch/Deutsch gemischt) – i18n steht noch aus

---

## 🏗️ Architektur

```
BACKEND:  py/ihco_filter.py
          • factions.json.gz      → IHCO Systeme extrahieren (Quelle der Wahrheit)
          • systems_1week.json.gz → updateTime Delta-Merge
          • ihco_systems.json     → Datenbank (im Repo, versioniert)
          • Verlorene Systeme werden erkannt und geloggt (⚠️)
          • Neue Systeme werden erkannt und geloggt (✅)

BUILD:    py/embed_map.py
          • ihco_map.html + .css + .js + ihco_systems.json
          • → index.html (alles embedded, portable Einzeldatei)

FRONTEND: ihco_map.html / ihco_map.css / ihco_map.js
          • Three.js r128 (CDN: cdnjs.cloudflare.com)
          • Orbit Controls manuell implementiert
          • Glow-Partikel mit RadialGradient + AdditiveBlending
          • Raycaster für Hover (threshold dynamisch)
          • Suche mit Autocomplete
          • Toggle: Kontrolliert / Präsent / Grid
          • DON'T PANIC Flash beim Laden 🙂
          • SVG-Overlay für Nachbar-Verbindungslinien

CONFIG:   ihco_config.js
          • Zoom: minRadius, maxRadius, zoomSpeed
          • Grid: gridVisible, gridSize1, gridSize2
          • Sterne: matControlledSize, matPresentSize, starHighlightSize
          • Labels: labelNeighborRadius, labelMaxZoomRadius, labelMaxNeighbors
          • Hover: hoverThresholdFactor (TODO: in Config aufnehmen)

KLASSEN:  js/stars/stars.js
          • BackgroundStars – dekorative Hintergrundsterne
          • SystemLayer – IHCO-Systeme, Toggle, Highlights, getNeighbors()
          js/grid/grid.js
          • InfiniteGrid – anti-aliased Shader, Labels, Toggle

WORKFLOWS:
          • update.yml  → Dienstags 06:00 UTC + manuell
                          Download → Filter → Merge → Commit
          • deploy.yml  → Bei jedem Push auf main
                          embed_map.py → index.html → GitHub Pages
```

---

## 📊 Datenstruktur – ihco_systems.json

```json
[
  {
    "name": "Intergalactic Hitchhiker's Coalition",
    "systems": [
      {
        "systemName": "Arietis Sector FW-W c1-11",
        "systemId64": 123456789,
        "isControllingFaction": true,
        "coords": { "x": 1.0, "y": 2.0, "z": 3.0 },
        "updateTime": "2026-05-20T12:00:00"
      }
    ]
  }
]
```

---

## 🗺️ BGS-Entscheidungsmatrix

| Systemstatus | IHCO Präsenz | Aktion |
|-------------|--------------|--------|
| 🌑 Unkolonisiert | – | 🚀 Kolonisierungsziel |
| 🤖 NPC kontrolliert | ✅ präsent | ⚔️ Konflikt auslösen |
| 🤖 NPC kontrolliert | ❌ nicht präsent | 📍 Expansionsziel |
| 🏴 Spielerfraktion kontrolliert | ✅ präsent | ⚠️ Squadleitung entscheidet |
| 🏴 Spielerfraktion kontrolliert | ❌ nicht präsent | ⚠️ Squadleitung entscheidet |
| 🟢 IHCO kontrolliert | ✅ | 🛡️ Verteidigen |

> ⚠️ **Regel:** Keine Konflikte oder aggressive Expansion gegen Spielerfraktionen ohne explizite Entscheidung der Squadleitung!

---

## 🗓️ Roadmap

### Kurzfristig
- [x] GitHub Actions live testen (manuell triggern)
- [ ] Repo aufräumen (Duplikate, veraltete Dateien)
- [ ] `hoverThresholdFactor` in `ihco_config.js` aufnehmen
- [ ] `labelMaxNeighbors` in `ihco_config.js` aufnehmen
- [ ] i18n Struktur anlegen (`locales/en.json`, `locales/de.json`)
- [ ] UI-Texte externalisieren

### Mittelfristig
- [ ] Cluster-Definition (manuell per JSON-Konfiguration)
- [ ] Fremdfraktionen in Clustern anzeigen (aus `factions.json.gz`)
- [ ] Spielerfraktionen kennzeichnen (`isPlayer: true` in Spansh)
- [ ] BGS-Tracker: Einfluss % pro System
- [ ] Aktionsempfehlungen pro System (aus BGS-Matrix)

### Langfristig
- [ ] Kolonisations-Planer (Hops, Warenlisten, Fortschritt)
- [ ] Hop-Routen visualisieren auf der Karte
- [ ] Federal Corvette kaufen 😄

---

## 💡 Wichtige Kontextinfos

- **Hauptstützpunkt:** Arietis Sector FW-W c1-11 – Punk Platform
- **Bestes Transportschiff:** HSS Colonel Hathi (Panther Clipper Mk II, 1.201t)
- **Explorations-Scout:** HSS Herr Mannelig (Mandalay) – ggf. unterwegs
- **Entfernte Bubbles:** ~120 Lj (2x, realistisch anbindbar per Daisy-Chain)
- **Sehr entfernte Bubble:** ~1.400 Lj (langfristiges Squadprojekt)
- **Entwicklungsumgebung:** Eclipse IDE mit PyDev, Windows, Python 3.14
- **Copilot Logbook:** https://github.com/vonstackelberg-hell/claude-cantina

---

## 🔗 Wichtige Links

| Ressource | URL |
|-----------|-----|
| Repo (ihco-guide) | https://github.com/vonstackelberg-hell/ihco-guide |
| Repo (claude-cantina) | https://github.com/vonstackelberg-hell/claude-cantina |
| Live | https://vonstackelberg-hell.github.io/ihco-guide/ |
| Spansh Dumps | https://spansh.co.uk/dumps |
| factions.json.gz | https://downloads.spansh.co.uk/factions.json.gz |
| systems_1week.json.gz | https://downloads.spansh.co.uk/systems_1week.json.gz |
| EDSM | https://www.edsm.net |
| Inara | https://inara.cz |

---

## 📝 Session-Notizen

### 2026-05-25 – Erste Engineer Session
- GitHub Pages eingerichtet
- Pipeline von lokal auf GitHub Actions migriert
- HTML/CSS/JS getrennt, embed_map.py angepasst
- Delta-Merge Logik mit Verloren-Erkennung implementiert
- Glow-Partikel mit AdditiveBlending eingeführt
- BGS-Entscheidungsmatrix definiert
- i18n als Anforderung aufgenommen
- Claude Cantina als separates Repo für Copilot-Logbücher geplant

### 2026-05-26 – Session 2: Grid & Pipeline
- GitHub Actions `update.yml` live getestet → ✅ grün
- `deploy.yml` manuell getriggert → ✅ grün
- Pipeline vollständig funktionsfähig (local → update → deploy → live)
- InfiniteGridHelper (anti-aliased Shader, Fyrestar-Technik) implementiert
  - XZ-Ebene, feines Gitter 100 Ly, grobes 500 Ly
  - Koordinaten-Labels als Canvas-Sprites (x,z in Ly)
  - Eigener Toggle-Button `btnGrid`
  - Grid standardmäßig eingeblendet
- `ihco_config.js` eingeführt (Zoom, Grid, Hover)
- `embed_map.py` um Config-Einbettung erweitert

### 2026-05-27 – Session 3: OOP-Refactoring & Nachbarlabels
- Branch `feature/oop-refactoring` erstellt und nach main gemerged
- `stars.js` mit Klassen `BackgroundStars` und `SystemLayer`
- `grid.js` mit Klasse `InfiniteGrid`
- `ihco_map.js` deutlich verschlankt, keine globalen Punktvariablen mehr
- `sizeAttenuation: false` für alle Sterne – schärfer beim Zoom
- Hover-Threshold dynamisch: `spherical.radius * factor`
- Raycaster ignoriert unsichtbare Gruppen (Toggle-Bug behoben)
- SVG-Overlay für Verbindungslinien implementiert
- `#hover`-Div erweitert: gehoverten Stern + Nachbarliste
- Linien führen von jedem Namenseintrag direkt zu seinem Stern
- `getNeighbors()` filtert nach Sichtbarkeit und `labelMaxNeighbors`
- Linien werden beim Zoomen neu berechnet (`lastHovered`)

---
*Zuletzt aktualisiert: 2026-05-27 – Session mit Claude Sonnet 4.6*
