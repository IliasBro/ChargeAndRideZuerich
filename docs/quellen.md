# Quellen und Daten: Charge & Ride Züri

## Datenquellen

### OpenStreetMap

Lizenz: Open Database License (ODbL)
Zugriff: OSMnx Python-Library
Tags verwendet: amenity=charging_station, amenity=restaurant, amenity=cafe, railway=station
URL: https://www.openstreetmap.org

### Velozählstellen Stadt Zürich

Datensatz: Daten der automatischen Fuss- und Velozählung, Viertelstundenwerte
URL Frequenzdaten: https://data.stadt-zuerich.ch/dataset/ted_taz_verkehrszaehlungen_werte_fussgaenger_velo
URL Standorte: https://www.stadt-zuerich.ch/geodaten/download/Standorte_der_automatischen_Fuss__und_Velozaehlungen
Parquet-Download 2025: https://data.stadt-zuerich.ch/dataset/ted_taz_verkehrszaehlungen_werte_fussgaenger_velo/download/2025_verkehrszaehlungen_werte_fussgaenger_velo.parquet
Lizenz: Creative Commons CCZero
Abgerufen: [Datum eintragen]

### Velozählstellen Kanton Zürich (OGD Dataset 3062)

Beschreibung: 97 Velozählstellen im ganzen Kanton Zürich, Optosensoren (LWL), seit 2016
URL Datenkatalog: https://www.zh.ch/de/politik-staat/statistik-daten/datenkatalog.html#/datasets/3062@tiefbauamt-kanton-zuerich
URL GIS-Browser: https://geo.zh.ch/maps?initialMapIds=TBAVMSZH
Lizenz: Open Government Data Kanton Zürich (CC BY 4.0)
Abgerufen: [Datum eintragen]

### SchweizMobil-Velorouten Kanton Zürich

Beschreibung: Offizielle Velo- und Mountainbikerouten, inkl. Velonetzplan
URL: https://data.stadt-zuerich.ch/dataset/ktzh_schweizmobil_routen__ogd_
Lizenz: Open Government Data
Abgerufen: [Datum eintragen]

### Gemeindegrenzen und Kantonsgrenze

Quelle: OSMnx (ox.geocode_to_gdf) oder swisstopo
URL swisstopo: https://www.swisstopo.admin.ch
Lizenz: Open use, Quellenangabe erforderlich

---

## Software und Tools

| Tool | Version | Quelle |
|---|---|---|
| Python | 3.11+ | https://python.org |
| GeoPandas | 0.14+ | https://geopandas.org |
| OSMnx | 1.9+ | Boeing (2017), siehe Literatur |
| pyarrow | 14.0+ | https://arrow.apache.org |
| scipy | 1.11+ | https://scipy.org |
| folium | 0.15+ | https://python-visualization.github.io/folium |
| contextily | 1.4+ | https://contextily.readthedocs.io |
| QGIS | 3.40 LTR | https://qgis.org |

---

## Literatur

Boeing, G. (2017). OSMnx: New Methods for Acquiring, Constructing, Analyzing, and Visualizing Complex Street Networks. Computers, Environment and Urban Systems, 65, 126-139.

Cliquet, G. (Hrsg.) (2006). Geomarketing: Methods and Strategies in Spatial Marketing. ISTE.

QGIS Development Team (2024). QGIS User Guide, Release 3.40. https://docs.qgis.org

Kanton Zürich, Amt fuer Mobilitaet (2025). Velozähldaten als OGD-Datensatz. https://www.zh.ch/de/mobilitaet/veloverkehr/veloverkehrsplanung/datenrundlagen-veloverkehr.html
