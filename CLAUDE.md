# CLAUDE.md - Projektkontext fuer Claude Code

Diese Datei wird von **Claude Code** automatisch gelesen, wenn du im Repository arbeitest.
Sie gibt dem KI-Assistenten den noetigen Kontext, um dir effizient helfen zu koennen.

---

## Projektueberblick

**Projektname:** Charge & Ride Zueri
**Ziel:** Identifikation der 5 besten Standorte fuer neue oeffentliche E-Bike-Ladestationen
im Kanton Zuerich mittels Geomarketing-Scoring-Modell.
**Kontext:** Semesterprojekt ZHAW, Modul Geomarketing & Spatial Data Analysis (FS 2026),
Dozent: Dr. Mario Gellrich.
**Deadline:** 27.05.2026
**Sprache:** Deutsch (Code-Kommentare und Dokumentation auf Deutsch)

---

## Projektstruktur und Aufgabenverteilung

```
notebooks/01_fetch_osm_zh.ipynb         -> OSM-Daten laden (OSMnx)
notebooks/02_process_zh_opendata.ipynb  -> Velozaehldaten verarbeiten
notebooks/03_scoring_model.ipynb        -> Scoring-Modell und Top-5-Ausgabe
data/raw/                               -> Rohdaten (nicht in Git, nur lokal)
data/processed/                         -> Verarbeitete GeoJSONs (in Git)
qgis_project/                           -> QGIS-Projektdatei
```

---

## Tech Stack

- Python 3.11+
- OSMnx: OpenStreetMap-Daten herunterladen und verarbeiten
- GeoPandas: Geodatenanalyse (Spatial Joins, Buffer, etc.)
- Shapely: Geometrie-Operationen
- pyproj: Koordinatentransformationen (WGS84 <-> LV95/EPSG:2056)
- Scipy: Kernel Density Estimation (KDE) fuer die Velo-Heatmap
- Folium: Interaktive HTML-Karten als Output
- Contextily: Basemaps fuer matplotlib-Plots
- Requests: HTTP-Calls zur Kanton ZH Open Data API
- QGIS 3.40 LTR: Finale Kartenerstellung und Layouting

---

## Datenquellen (verifiziert, alle OGD/kostenlos)

### PRIMAER: Kantonale Velozaehlstellen (97 Stellen im ganzen Kanton ZH!)

Dies ist die Hauptdatenquelle fuer Velofrequenzen. Das Kanton ZH hat seit 2016
ein eigenes kantonales Messnetz mit 97 aktiven Zaehlstellen im ganzen Kanton.
Die Daten sind als OGD (Open Government Data) verfuegbar.

OGD-Datensatz Kanton ZH (Dataset ID 3062):
https://www.zh.ch/de/politik-staat/statistik-daten/datenkatalog.html#/datasets/3062@tiefbauamt-kanton-zuerich

GIS-Browser mit allen Velomessstellen (visuell):
https://geo.zh.ch/maps?initialMapIds=TBAVMSZH

Velozaehldatenzentrale Schweiz (alle CH-Zaehlstellen, inkl. ZH):
https://velo-ch.eco-counter.com/

### SEKUNDAER: Stadt Zuerich Velozaehlstellen (25 Stellen, nur Stadtgebiet)

Diese Daten sind sehr detailliert (Viertelstundenwerte seit 2009).

Standorte als GeoJSON (direkt downloadbar, EPSG:4326):
https://www.stadt-zuerich.ch/geodaten/download/Standorte_der_automatischen_Fuss__und_Velozaehlungen
WFS-Schnittstelle fuer direkten API-Zugriff ebenfalls verfuegbar.

Frequenzdaten (CSV, taeglich aktualisiert, CC-Zero Lizenz):
https://data.stadt-zuerich.ch/dataset/ted_taz_verkehrszaehlungen_werte_fussgaenger_velo

Direkter CSV-Download 2025:
https://data.stadt-zuerich.ch/dataset/ted_taz_verkehrszaehlungen_werte_fussgaenger_velo/download/2025_verkehrszaehlungen_werte_fussgaenger_velo.csv

Join-Schluessel: FK_ZAEHLER verbindet Standorte-GeoJSON mit Frequenz-CSV.
Filterung: Nur Zeilen mit Typ='Velo' verwenden (nicht Fussgaenger).

### BONUS: SchweizMobil-Velorouten Kanton ZH (OGD!)

Offizielle Velo-, Mountainbike- und Skatingrouten des ganzen Kantons ZH.
Enthaelt auch den Velonetzplan. Verfuegbar als Shapefile, GeoPackage, GeoJSON.
Besser als OSMnx fuer offizielle Routen!

https://data.stadt-zuerich.ch/dataset/ktzh_schweizmobil_routen__ogd_

### OpenStreetMap (via OSMnx) - fuer POIs

```python
import osmnx as ox
place = "Kanton Zuerich, Switzerland"
# Ladestationen: amenity=charging_station
# Gastronomie:   amenity=restaurant / amenity=cafe
# Bahnhoefe:     railway=station
```

### Gemeindegrenzen Kanton Zuerich

```python
kanton_zh = ox.geocode_to_gdf("Kanton Zuerich, Switzerland")
```

---

## Koordinatensysteme

- OSM-Eingabedaten: WGS84 (EPSG:4326)
- Kantonale ZH-Daten: CH1903+ / LV95 (EPSG:2056)
- Arbeits-CRS im Projekt: LV95 (EPSG:2056) -- immer fuer Distanzberechnungen in Metern!

```python
gdf = gdf.to_crs(epsg=2056)   # -> LV95 fuer Berechnungen
gdf = gdf.to_crs(epsg=4326)   # -> WGS84 fuer Folium-Karten
```

---

## Workflow der Notebooks

### Notebook 01 - 01_fetch_osm_zh.ipynb

Aufgabe: Alle OSM-Daten fuer Kanton Zuerich laden und als GeoJSON speichern.

Output-Files in data/processed/:
- ladestationen_zh.geojson
- gastronomie_zh.geojson
- bahnhoefe_zh.geojson
- velorouten_zh.geojson

### Notebook 02 - 02_process_zh_opendata.ipynb

Aufgabe: Velozaehldaten laden, bereinigen, mittlere Tagesfrequenz pro Zaehlstelle berechnen.

Output-Files in data/processed/:
- velozaehlstellen_mit_frequenz.geojson

### Notebook 03 - 03_scoring_model.ipynb

Aufgabe: Scoring-Modell ausfuehren, Top-5-Standorte berechnen, Karten exportieren.

Scoring-Logik:
```python
score = (
    w1 * normalized_velo_frequency +      # Hohe Frequenz = gut
    w2 * normalized_distance_to_charger   # Weit weg von Ladesaeule = gut
)
# Gewichtung initial: w1=0.5, w2=0.5 (anpassbar, begruendet dokumentieren)
# POIs innerhalb Versorgungszone (3km-Puffer) werden ausgefiltert
```

Output-Files in data/processed/:
- top5_standorte.geojson
- alle_kandidaten_scored.geojson

---

## Code-Standards

- Kommentare: Auf Deutsch, klar und verstaendlich
- CRS: Immer explizit setzen und pruefen mit gdf.crs
- Datei-Speicherung: Immer in data/processed/ mit sprechendem Namen
- Keine hardcodierten Pfade: pathlib.Path verwenden
- Reproduzierbarkeit: Jedes Notebook muss von oben nach unten fehlerfrei durchlaufen

Standard-Import-Block fuer jedes Notebook:

```python
from pathlib import Path
import geopandas as gpd
import pandas as pd
import osmnx as ox
import matplotlib.pyplot as plt

ROOT = Path("..").resolve()
DATA_RAW = ROOT / "data" / "raw"
DATA_PROCESSED = ROOT / "data" / "processed"
DATA_PROCESSED.mkdir(parents=True, exist_ok=True)
```

---

## Bekannte Fallstricke

1. OSMnx Rate Limits: Nicht in einer Schleife ohne time.sleep() abfragen.
2. CRS-Mismatch: Vor jedem Spatial Join CRS beider GDFs pruefen.
3. Stadt ZH CSV hat VIERTELSTUNDENWERTE (96 pro Tag) -> fuer Tagesfrequenz summieren.
4. CSV enthaelt Fuss- UND Velozaehlungen gemischt -> nach Geraetetyp filtern!
5. Join-Schluessel: FK_ZAEHLER im GeoJSON = ZAEHLER_ID im CSV.
6. Grosse OSM-Downloads: Einmal laden, als GeoJSON cachen.
7. QGIS-Pfade: Das .qgz-File nutzt relative Pfade.

---

## Bewertungskriterien (Code-Anteil, 50% der Gesamtnote)

- Technische Umsetzung und Standards: 15%
- Projekt- und Repository-Organisation: 5%
- Dokumentation (README + Kommentare): 10%
- Reproduzierbarkeit und Setup: 5%
- Versionierung und Git-Praxis: 5%

Sauberer, kommentierter, reproduzierbarer Code ist genauso wichtig wie korrekte Ergebnisse!
