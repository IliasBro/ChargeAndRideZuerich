# Machbarkeitsanalyse: Charge & Ride Züri

Stand: FS2026

## Gesamtbewertung: Umsetzbar

Die Projektidee ist solide konzipiert und vollständig umsetzbar.
Alle benötigten Daten sind frei verfügbar. Die Datenlage ist
besser als in der ursprünglichen Projektidee angenommen.

---

## Datenverfügbarkeit

### Kantonale Velozählstellen (Kanton Zürich, Dataset 3062)

Status: Verfügbar, besser als ursprünglich angenommen.

97 aktive Velozählstellen im ganzen Kanton Zürich, nicht nur im Stadtgebiet.
Das Netz besteht seit 2016. Technologie: Optosensoren (LWL), Genauigkeit über 95%.
Die Daten sind als Open Government Data (OGD) kostenlos verfügbar.

- OGD-Datensatz: zh.ch Datenkatalog, Dataset ID 3062, Tiefbauamt Kanton ZH
- GIS-Browser: https://geo.zh.ch/maps?initialMapIds=TBAVMSZH
- Schweizweite Übersicht: https://velo-ch.eco-counter.com/

Dies widerlegt die ursprüngliche Annahme, dass flächendeckende Zähldaten nur für die Stadt Zürich verfügbar seien. Hypothese 3 der Projektidee ist damit deutlich stärker begründet.

### Velozählstellen Stadt Zürich

Status: Verfügbar, sehr detailliert.

25 Zählstellen mit Viertelstundenwerten seit 2009. Abdeckung: ausschliesslich Stadtgebiet.

- Standorte als GeoJSON, Shapefile, GeoPackage: https://www.stadt-zuerich.ch/geodaten/download/Standorte_der_automatischen_Fuss__und_Velozaehlungen
- Frequenzdaten als CSV und Parquet (CC-Zero, täglich aktualisiert): https://data.stadt-zuerich.ch/dataset/ted_taz_verkehrszaehlungen_werte_fussgaenger_velo
- Join-Schlüssel: FK_ZAEHLER

Zu beachten: Das CSV/Parquet enthält Viertelstundenwerte (96 pro Tag) und kombiniert
Fuss- und Velozählungen. Im Code muss nach Typ gefiltert und die Tagesfrequenz
korrekt durch Summierung aller Viertelstunden berechnet werden.

### SchweizMobil-Velorouten Kanton Zürich

Status: Verfügbar, nützliche Ergänzung.

Offizielle Velo- und Mountainbikerouten als OGD in mehreren Formaten (Shapefile, GeoJSON, GeoPackage).
Besser geeignet für offizielle Routennetze als reine OSMnx-Daten.

Download: https://data.stadt-zuerich.ch/dataset/ktzh_schweizmobil_routen__ogd_

### OpenStreetMap via OSMnx

Status: Verfügbar, kein API-Key erforderlich.

Ladestationen (amenity=charging_station): verfügbar, gut getaggt.
Gastronomie (restaurant, cafe): sehr gut abgedeckt im Kanton ZH.
Bahnhöfe (railway=station): vollständig verfügbar.

### Gemeindegrenzen

Status: Verfügbar.

Direkt via OSMnx (ox.geocode_to_gdf) oder von swisstopo kostenlos beziehbar.

---

## Technische Machbarkeit

### OSMnx

Download des Kantons ZH dauert einmalig ca. 5-10 Minuten.
Caching über ox.settings.use_cache=True vermeidet Wiederholungen.
Keine Authentifizierung erforderlich.

### GeoPandas Spatial Operations

Buffer (3 km): Standard-Operation, keine besonderen Anforderungen.
Spatial Join (Point-in-Polygon): Standard-Operation.
KDE via scipy.stats.gaussian_kde: bewährt für Heatmaps aus Punktdaten.

### Parquet-Format (Stadt ZH Frequenzdaten)

Das Parquet-Format ist 5-10x schneller zu laden als CSV und benötigt weniger Arbeitsspeicher.
Voraussetzung: pyarrow-Paket (in requirements.txt enthalten).
pandas kann Parquet direkt von einer URL laden: pd.read_parquet(url)

### QGIS 3.40

GeoJSON-Import: problemlos, Standard-Workflow.
Layout-Erstellung: wie in der Vorlesung behandelt.

---

## Strategie: Verwendung beider Velozähldatensätze

Die Analyse kombiniert beide verfügbaren Datensätze:

Die kantonalen Daten (97 Zählstellen) bilden die Grundlage für die
flächendeckende Nachfrageanalyse über den gesamten Kanton. Sie decken
die für das Projekt besonders relevanten Bereiche ausserhalb des Stadtzentrums ab
(Albispass, Pfannenstiel, Glattal, Limmattal).

Die Stadtdaten (25 Zählstellen) ergänzen die Analyse im Stadtgebiet mit
höherer zeitlicher Auflösung und einer längeren Messreihe (seit 2009).

Beide Datensätze werden über den Join-Schlüssel FK_ZAEHLER mit den
Standort-GeoJSONs verbunden und zu einem gemeinsamen GeoDataFrame zusammengeführt.

---

## Empfohlene Anpassungen gegenüber der ursprünglichen Projektidee

1. Kantonale Zähldaten als primäre Datengrundlage verwenden, nicht nur Stadtdaten.
   Die ursprüngliche Limitation (nur Stadtgebiet) besteht nicht mehr.

2. Scoring-Gewichtungen begründen und dokumentieren.
   Sensitivitätsanalyse mit mindestens zwei Gewichtungsvarianten zeigen.

3. Bahnhofs-Bonus im Scoring begründen:
   Bahnhöfe sind für Pendler-Standorte relevanter als Gastronomie.

---

## Zeitplan bis Deadline 27.05.2026

| Woche  | Aufgabe |
|--------|---------|
| KW 16-17 | Notebook 01: OSM-Daten laden und prüfen |
| KW 17-18 | Notebook 02: Velodaten beider Quellen integrieren |
| KW 18-19 | Notebook 03: Scoring-Modell implementieren |
| KW 19-20 | QGIS: drei Karten layouten |
| KW 20-21 | Präsentation erstellen und Video aufnehmen |
| KW 21    | Finales Review, Commit, Upload auf Moodle |
