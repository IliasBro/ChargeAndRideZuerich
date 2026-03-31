# Charge & Ride Züri
## Geodatenbasierte Standortoptimierung für E-Bike-Ladestationen im Kanton Zürich

Modul: Geomarketing & Spatial Data Analysis, ZHAW School of Management and Law
Semester: FS 2026
Dozent: Dr. Mario Gellrich
Abgabe: 27.05.2026

---

## Projektbeschreibung

Der Kanton Zürich verzeichnet ein starkes Wachstum beim E-Bike-Verkehr, sowohl im Pendlerbereich als auch im Freizeitsport. Das öffentliche Ladenetz hinkt jedoch hinterher, insbesondere ausserhalb der Stadtzentren.

Dieses Projekt entwickelt ein datengestütztes Geomarketing-Scoring-Modell, das die 5 optimalen Standorte für neue öffentliche E-Bike-Ladestationen im Kanton Zürich identifiziert.

### Forschungsfrage

Wo im Kanton Zürich überschneiden sich hohe Velo-Frequenzen (Pendler/Freizeit) und ideale Verweilinfrastruktur (Bahnhöfe/Gastronomie), ohne dass bereits ausreichende Ladeinfrastruktur vorhanden ist?

### Hypothesen

1. Beliebte Freizeithügelzüge (Albispass, Pfannenstiel) haben hohe Gastrodichte, aber kaum Ladestationen.
2. Grosse ÖV-Knotenpunkte in der Agglomeration (Uster, Dietikon) bieten Potenzial für "Park & Charge"-Konzepte.
3. Kantonale Velozähldaten ermöglichen eine präzisere Standortwahl als reine OSM-Routendaten.

---

## Repository-Struktur

```
zh-ebike-charging/
    data/
        raw/                    Rohdaten (nicht versioniert, siehe .gitignore)
        processed/              Verarbeitete GeoJSONs fuer QGIS und Analyse
    notebooks/
        01_fetch_osm_zh.ipynb           OSM-Daten via OSMnx laden
        02_process_zh_opendata.ipynb    Velozaehldaten verarbeiten
        03_scoring_model.ipynb          Scoring-Modell und Top-5-Ergebnis
    qgis_project/
        zh_ebike_analysis.qgz           QGIS-Projektdatei
    docs/
        machbarkeitsanalyse.md          Machbarkeitspruefung
        quellen.md                      Quellenverzeichnis
    CLAUDE.md                           Kontext fuer Claude Code
    requirements.txt
    README.md
```

---

## Datenquellen und Strategie

Das Projekt nutzt zwei komplementäre Velozähldatensätze, die unterschiedliche räumliche Abdeckungen bieten. Beide sind als Open Government Data (OGD) kostenlos verfügbar.

### Velozähldaten: zwei Datensätze, ein gemeinsames Ziel

Das Kanton Zürich hat seit 2016 ein eigenes kantonales Messnetz mit 97 aktiven Zählstellen, die den ganzen Kanton abdecken. Zusätzlich betreibt die Stadt Zürich seit 2009 ein feineres Netz mit 25 Stationen ausschliesslich im Stadtgebiet.

| Merkmal | Stadt Zürich | Kanton Zürich |
|---|---|---|
| Zählstellen | 25 | 97 |
| Abdeckung | Nur Stadtgebiet | Ganzer Kanton |
| Verfügbar seit | 2009 | 2016 |
| Granularität | Viertelstunden | Stunden/Tage |
| Format | CSV, Parquet, GeoJSON | CSV (manueller Download) |

Die Strategie im Projekt: Die kantonalen Daten bilden die Grundlage für die flächendeckende Nachfrageanalyse über den gesamten Kanton (relevant für Standorte in der Agglomeration und auf den Freizeitrouten). Die Stadtdaten werden zusätzlich für das Stadtgebiet eingesetzt, da sie durch die höhere zeitliche Auflösung und längere Messreihe eine genauere Heatmap im dicht besiedelten Bereich ermöglichen. Beide Datensätze werden über denselben Schlüssel (FK_ZAEHLER) mit den georeferenzierten Standortdaten verbunden und zu einem gemeinsamen GeoDataFrame zusammengeführt.

### Datenquellen im Detail

**Velozählstellen Stadt Zürich**
- Standorte (GeoJSON, Shapefile, GeoPackage u.a.): https://www.stadt-zuerich.ch/geodaten/download/Standorte_der_automatischen_Fuss__und_Velozaehlungen
- Frequenzdaten (Parquet, täglich aktualisiert, CC-Zero): https://data.stadt-zuerich.ch/dataset/ted_taz_verkehrszaehlungen_werte_fussgaenger_velo
- Parquet-Direktdownload 2025: https://data.stadt-zuerich.ch/dataset/ted_taz_verkehrszaehlungen_werte_fussgaenger_velo/download/2025_verkehrszaehlungen_werte_fussgaenger_velo.parquet

**Velozählstellen Kanton Zürich (OGD Dataset 3062)**
- Datenkatalog: https://www.zh.ch/de/politik-staat/statistik-daten/datenkatalog.html#/datasets/3062@tiefbauamt-kanton-zuerich
- GIS-Browser zur Vorschau: https://geo.zh.ch/maps?initialMapIds=TBAVMSZH
- Manueller Download erforderlich, Datei in data/raw/ ablegen.

**OpenStreetMap via OSMnx**
- Bestehende E-Bike-Ladestationen (amenity=charging_station)
- Gastronomie-POIs (amenity=restaurant, amenity=cafe)
- Bahnhöfe (railway=station)
- Kantonsgrenze Zürich

**SchweizMobil-Velorouten Kanton Zürich (OGD)**
- Offizielle Velo- und Mountainbikerouten als GeoJSON/Shapefile
- https://data.stadt-zuerich.ch/dataset/ktzh_schweizmobil_routen__ogd_

---

## Setup und Installation

Voraussetzungen: Python 3.11 oder neuer, Git, QGIS 3.40 LTR

```bash
# Repository klonen
git clone https://github.com/EUER-USERNAME/zh-ebike-charging.git
cd zh-ebike-charging

# Virtuelle Umgebung erstellen und aktivieren
python -m venv .venv
source .venv/bin/activate      # Mac und Linux
# .venv\Scripts\activate       # Windows

# Abhaengigkeiten installieren
pip install -r requirements.txt

# Jupyter starten
jupyter lab notebooks/
```

---

## Workflow

Die Notebooks sind in Reihenfolge auszuführen:

Notebook 01 laedt alle OSM-Daten (Ladestationen, Gastronomie, Bahnhöfe, Kantonsgrenze) via OSMnx und speichert sie als GeoJSON in data/processed/.

Notebook 02 laedt die Velozähldaten beider Quellen (Stadt ZH via Parquet-URL, Kanton ZH nach manuellem Download), berechnet die mittlere Tagesfrequenz pro Zählstelle und speichert georeferenzierte GeoJSONs.

Notebook 03 führt das Scoring-Modell aus: Supply-Analyse (3-km-Puffer um bestehende Ladestationen), Demand-Analyse (KDE-Heatmap der Velofrequenzen), Punktebewertung aller POIs ausserhalb der Versorgungszone, Export der Top-5-Standorte als GeoJSON für QGIS.

---

## Methodik

**Supply-Analyse:** 3 km-Puffer um bestehende Ladestationen zeigen bereits versorgte Gebiete.

**Demand-Analyse:** Kernel Density Estimation (KDE) auf Basis der Velozählfrequenzen ergibt eine räumliche Heatmap der Nachfrage.

**Scoring-Modell:** Gastronomie-POIs und Bahnhöfe ausserhalb der Versorgungszone erhalten Punkte in Abhängigkeit von der Nähe zu Velo-Hotspots und der Distanz zur nächsten bestehenden Ladestation. Die Gewichtungen sind im Notebook dokumentiert und begründet.

---

## Team

| Name | Aufgabe |
|---|---|
| [Name 1] | Notebook 01: OSM Data Fetching |
| [Name 2] | Notebook 02: Open Data Processing |
| [Name 3] | Notebook 03: Scoring Model und QGIS |

---

## Lizenz

Die verwendeten Daten stehen unter Creative Commons CCZero (Open Government Data). Der eigene Code steht unter der MIT Lizenz.
