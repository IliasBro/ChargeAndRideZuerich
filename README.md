# Charge & Ride Zürich
### Geodatenbasierte Standortoptimierung für E-Bike-Ladestationen im Kanton Zürich

Modul: **Geomarketing & Spatial Data Analysis** | ZHAW School of Management and Law  
Semester: FS 2026 | Dozent: Dr. Mario Gellrich | Abgabe: 27.05.2026

---

## Projektbeschreibung

Der Kanton Zürich verzeichnet ein starkes Wachstum im E-Bike-Verkehr — sowohl im Pendlerbereich als auch im Freizeitsport. Das öffentliche Ladenetz hinkt jedoch hinterher, insbesondere ausserhalb der Stadtzentren.

Dieses Projekt entwickelt ein datengestütztes **Geomarketing-Scoring-Modell**, das die **5 optimalen Standorte** für neue öffentliche E-Bike-Ladestationen im Kanton Zürich identifiziert. Die Analyse kombiniert reale Velofrequenzdaten aus zwei kantonalen Messnetzen mit POI-Daten aus OpenStreetMap.

### Forschungsfrage

> Wo im Kanton Zürich überschneiden sich hohe Velo-Frequenzen und geeignete Verweilinfrastruktur (Bahnhöfe, Gastronomie), ohne dass bereits ausreichende Ladeinfrastruktur vorhanden ist?

### Hypothesen

1. Beliebte Freizeitdestinationen (Albispass, Pfannenstiel) haben hohe Gastrodichte, aber kaum Ladestationen.
2. Grosse ÖV-Knotenpunkte in der Agglomeration (Uster, Dietikon) bieten Potenzial für «Park & Charge»-Konzepte.
3. Kantonale Velozähldaten ermöglichen eine präzisere Standortwahl als reine OSM-Routendaten.

---

## Methodik

### Supply-Analyse
3 km-Puffer um bestehende Ladestationen definieren die bereits versorgte Zone («weisse Flecken» = unversorgte Gebiete).

### Demand-Analyse
Kernel Density Estimation (KDE) auf Basis der gewichteten Velozählfrequenzen (DTV = Durchschnittlicher Tagesverkehr) erzeugt eine räumliche Nachfrage-Heatmap.

### Scoring-Modell

Alle POIs (Gastronomie + Bahnhöfe) **ausserhalb** der Versorgungszone werden bewertet:

```
Score = w1 × norm(KDE-Velo-Score) + w2 × norm(Distanz zur nächsten Ladestation)
```

| Parameter | Wert | Begründung |
|---|---|---|
| w1 (Velofrequenz) | 0.5 | Hohe Nachfrage = guter Standort |
| w2 (Distanz) | 0.5 | Weisse Flecken priorisieren |
| Versorgungsradius | 3 km | Praxisüblicher E-Bike-Reichweitenpuffer |

---

## Datenquellen

Alle Datensätze sind **Open Government Data (OGD)**, kostenlos und ohne Registrierung verfügbar.

### Velozähldaten (Hauptquelle)

| | Stadt Zürich | Kanton Zürich |
|---|---|---|
| Zählstellen | 25 | 97 |
| Abdeckung | Nur Stadtgebiet | Ganzer Kanton |
| Granularität | Viertelstunden | Stunden |
| Format | CSV / Parquet | CSV |
| Lizenz | CC Zero | CC Zero |

- **Stadt ZH Standorte (GeoJSON):** [stadt-zuerich.ch Geodaten](https://www.stadt-zuerich.ch/geodaten/download/Standorte_der_automatischen_Fuss__und_Velozaehlungen)
- **Stadt ZH Frequenzen (Parquet):** [data.stadt-zuerich.ch](https://data.stadt-zuerich.ch/dataset/ted_taz_verkehrszaehlungen_werte_fussgaenger_velo)
- **Kanton ZH Frequenzen (OGD Dataset 3062):** [zh.ch Datenkatalog](https://www.zh.ch/de/politik-staat/statistik-daten/datenkatalog.html#/datasets/3062@tiefbauamt-kanton-zuerich)

### OpenStreetMap via OSMnx

- Bestehende E-Bike-Ladestationen (`amenity=charging_station`)
- Gastronomie-POIs (`amenity=restaurant`, `amenity=cafe`)
- Bahnhöfe (`railway=station`)
- Kantonsgrenze Zürich

---

## Repository-Struktur

```
ChargeAndRideZuerich/
├── notebooks/
│   ├── 01_fetch_osm_zh.ipynb            # OSM-Daten laden (OSMnx)
│   ├── 02_process_zh_opendata.ipynb     # Velozähldaten verarbeiten + DTV berechnen
│   └── 03_scoring_model.ipynb           # Scoring-Modell und Top-5-Ausgabe
├── data/
│   ├── raw/                             # Rohdaten (nicht versioniert, siehe .gitignore)
│   │   └── verkehrszaehldaten_velo_kanton_zh_2024.csv
│   └── processed/                       # Alle Output-GeoJSONs (versioniert)
│       ├── kanton_zh_boundary.geojson
│       ├── ladestationen_zh.geojson
│       ├── gastronomie_zh.geojson
│       ├── bahnhoefe_zh.geojson
│       ├── velozaehlstellen_stadt_zh.geojson
│       ├── velozaehlstellen_kanton_zh.geojson
│       ├── supply_zone_3km.geojson
│       ├── alle_kandidaten_scored.geojson
│       ├── top5_standorte.geojson
│       └── interaktive_karte.html
├── qgis_project/
│   └── charge_and_ride_zueri.qgz        # QGIS-Projektdatei (relative Pfade)
├── docs/
├── requirements.txt
├── CLAUDE.md
└── README.md
```

---

## Setup und Installation

**Voraussetzungen:** Python 3.11+, Git, QGIS 4.0

```bash
# Repository klonen
git clone https://github.com/<USERNAME>/ChargeAndRideZuerich.git
cd ChargeAndRideZuerich

# Virtuelle Umgebung erstellen und aktivieren
python -m venv .venv
source .venv/bin/activate        # Mac / Linux
# .venv\Scripts\activate         # Windows

# Abhängigkeiten installieren
pip install -r requirements.txt

# Jupyter starten
jupyter lab
```

### Rohdaten ablegen

Der Kanton-ZH-Datensatz muss einmalig manuell heruntergeladen werden:

1. [OGD Dataset 3062](https://www.zh.ch/de/politik-staat/statistik-daten/datenkatalog.html#/datasets/3062@tiefbauamt-kanton-zuerich) öffnen
2. CSV herunterladen
3. Datei als `data/raw/verkehrszaehldaten_velo_kanton_zh_2024.csv` ablegen

---

## Workflow

Die Notebooks müssen **in Reihenfolge** ausgeführt werden:

### Notebook 01 — OSM-Daten laden
Lädt Kantonsgrenze, Ladestationen, Gastronomie und Bahnhöfe via OSMnx aus OpenStreetMap.  
**Output:** `ladestationen_zh.geojson`, `gastronomie_zh.geojson`, `bahnhoefe_zh.geojson`, `kanton_zh_boundary.geojson`

### Notebook 02 — Velozähldaten verarbeiten
- **Teil A+B:** Stadt-ZH-Frequenzdaten via Parquet-URL laden, Viertelstundenwerte zu DTV aggregieren, mit Standort-Koordinaten (LV95 direkt im CSV) verbinden.
- **Teil C:** Kantonale Stundenwerte laden, alle Fahrtrichtungen pro Tag summieren, DTV berechnen, GeoDataFrame aus WGS84-Koordinaten erstellen.

**Output:** `velozaehlstellen_stadt_zh.geojson`, `velozaehlstellen_kanton_zh.geojson`  
**Schlüsselspalte:** `dtv_velos` (Durchschnittlicher Tagesverkehr Velos)

### Notebook 03 — Scoring-Modell
Supply-Analyse → KDE-Heatmap → Scoring → Top-5-Export.  
**Output:** `supply_zone_3km.geojson`, `alle_kandidaten_scored.geojson`, `top5_standorte.geojson`, `interaktive_karte.html`

---

## Ergebnisse

Die Top-5-Standorte sind in `data/processed/top5_standorte.geojson` gespeichert und können direkt in QGIS oder im Browser visualisiert werden:

```bash
# Interaktive Karte im Browser öffnen
open data/processed/interaktive_karte.html
```

Die finale Kartendarstellung wird mit QGIS 4.0 erstellt (`qgis_project/charge_and_ride_zueri.qgz`).

---

## Koordinatensysteme

| CRS | EPSG | Verwendung |
|---|---|---|
| WGS84 | 4326 | GeoJSON-Speicherung, Folium-Karten |
| LV95 | 2056 | Alle Distanzberechnungen in Metern |

---

## Lizenz

Verwendete Daten: **Creative Commons CC Zero** (Open Government Data Kanton und Stadt Zürich)  
Eigener Code: **MIT License**
