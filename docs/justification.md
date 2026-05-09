# Standortbegründung: Top-5 E-Bike-Ladestationen Kanton Zürich

**Projekt:** Charge & Ride Züri — ZHAW Semesterprojekt FS 2026  
**Modul:** Geomarketing & Spatial Data Analysis  
**Dozent:** Dr. Mario Gellrich  

---

## Executive Summary

Ziel der Analyse war die Identifikation der **5 optimalen Standorte** für neue öffentliche E-Bike-Ladestationen im Kanton Zürich — dort, wo gleichzeitig hohe Velofrequenz herrscht und bestehende Infrastruktur fehlt.

| Rang | Name | Ort (ca.) | Score | Stärke |
|------|------|-----------|-------|--------|
| 1 | **Freihof** | Kanton ZH Ost | 0.505 | Versorgungslücke |
| 2 | **NeftiHuus** | Kanton ZH Nordost | 0.517 | Versorgungslücke |
| 3 | **Hopp de Bäse!** | Kanton ZH Südost | 0.522 | Versorgungslücke |
| 4 | **Il Salento** | Kanton ZH Mitte/Süd | 0.505 | Velofrequenz |
| 5 | **Gartentropfen** | Kanton ZH Nordwest | 0.500 | Versorgungslücke |

---

## Methodik

### Kandidatenpool

Ausgangspunkt waren alle geeigneten **Points of Interest (POI)** aus OpenStreetMap:
- **1'928 Gastronomie-Standorte** (Restaurants, Cafés, Bars)
- **122 Bahnhöfe**

Als erster Filter wurden alle POIs **innerhalb der 3-km-Versorgungszone** der 11 bestehenden öffentlichen E-Bike-Ladestationen ausgeschlossen. Diese Zone deckt lediglich **14.3% des Kantonsgebiets** ab — der überwiegende Teil des Kantons ist heute unversorgt.

**Resultat: 2'050 bewertete Kandidaten** ausserhalb der bestehenden Versorgungszone.

---

### Scoring-Formel

```
Gesamtscore = 0.5 × norm_kde_score  +  0.5 × norm_dist_score
```

Beide Kriterien wurden mittels **Min-Max-Normalisierung** auf den Wertebereich [0, 1] skaliert, sodass sie gleichgewichtig in den Score einfliessen.

---

### Kriterium 1 — Velofrequenz (Gewicht: 50%)

**Warum:** Ladestationen entfalten den grössten Nutzen dort, wo viele Veloreisende unterwegs sind.

**Methodik — Kernel Density Estimation (KDE):**
- Datengrundlage: **121 Velozählstellen** (97 kantonal + 24 städtisch) mit validierten Jahresdaten 2024–2025
- Jede Zählstelle wurde mit ihrem **mittleren Tagesverkehr (DTV)** gewichtet
- Die KDE berechnet daraus eine kontinuierliche Velo-Heatmap über den gesamten Kanton
- Für jeden POI-Kandidaten wurde der KDE-Wert am jeweiligen Standort extrahiert

**Interpretation:** Ein hoher KDE-Score bedeutet, dass der Standort in einem Gebiet mit intensiver Velonutzung liegt.

---

### Kriterium 2 — Distanz zur nächsten Ladestation (Gewicht: 50%)

**Warum:** Neue Stationen sollen bestehende Angebote ergänzen, nicht konkurrenzieren.

**Methodik:**
- Berechnung der euklidischen Distanz von jedem Kandidaten zur nächstgelegenen der 11 bestehenden Ladestationen (LV95/EPSG:2056)
- **Grössere Distanz = höherer Score** (mehr Versorgungslücke = attraktiverer Standort)

---

### Greedy-Algorithmus mit Ausschlusszone

Die Top-5 wurden **iterativ** ausgewählt:

```
Iteration 1: Kandidat mit höchstem Score → Rang 1
             Alle Kandidaten im 3-km-Radius ausgeschlossen
             Distanzen neu berechnet (inkl. neu gewählter Station)
Iteration 2: Nächstbester Kandidat → Rang 2
             ...
```

Dieser Ansatz verhindert räumliche Clusterbildung und gewährleistet, dass die 5 Standorte den Kanton **flächendeckend** abdecken.

---

## Die Top-5 Standorte im Detail

### Rang 1 — Freihof

| Attribut | Wert |
|----------|------|
| Typ | Gastronomie |
| Koordinaten | 47.5219°N, 8.8900°E |
| Gesamtscore | 0.5046 |
| Distanz zu nächster Station | **25.09 km** |
| KDE-Score (norm.) | 0.0092 |

**Begründung:** Der Freihof liegt in einer der grössten Versorgungslücken des Kantons — über 25 km von der nächstgelegenen Ladestation entfernt. Diese extreme geografische Isolation macht ihn zur prioritären Wahl. Das Lokal dient als natürlicher Rastpunkt für Tourenvelofahrer, die in diesen ländlichen Regionen unterwegs sind. Der vergleichsweise niedrige KDE-Score spiegelt die geringere Zähldichte im ländlichen Raum wider, sagt aber nichts über das tatsächliche Velopotenzial dieser Routengebiete aus.

---

### Rang 2 — NeftiHuus

| Attribut | Wert |
|----------|------|
| Typ | Gastronomie |
| Koordinaten | 47.5266°N, 8.6665°E |
| Gesamtscore | 0.5174 |
| Distanz zu nächster Station | **16.84 km** |
| KDE-Score (norm.) | 0.0348 |

**Begründung:** Das NeftiHuus kombiniert eine signifikante Versorgungslücke (knapp 17 km) mit einem etwas höheren Frequenz-Score als Rang 1. Der Standort im nordöstlichen Kantonsgebiet erschliesst ein Gebiet, das bisher komplett ohne E-Bike-Ladeinfrastruktur ist. Als Gastronomie-Betrieb bietet er eine ideale Rastpause-Infrastruktur (Sitzplätze, WC, Verpflegung), die Veloreisende ohnehin anziehen.

---

### Rang 3 — Hopp de Bäse!

| Attribut | Wert |
|----------|------|
| Typ | Gastronomie |
| Koordinaten | 47.4052°N, 8.7254°E |
| Gesamtscore | 0.5215 |
| Distanz zu nächster Station | **14.21 km** |
| KDE-Score (norm.) | 0.0431 |

**Begründung:** Rang 3 weist den höchsten Gesamtscore der Top 5 auf, da er Versorgungslücke und Frequenz am ausgewogensten kombiniert. Mit 14 km Distanz zur nächsten Station und dem höchsten KDE-Score unter den "Lücken-Standorten" repräsentiert er das ideale Verhältnis aus Nachfragepotenzial und Unterversorgung. Der südöstliche Teil des Kantons gewinnt als Velodestination (z.B. Zürichsee-Routen) zunehmend an Bedeutung.

---

### Rang 4 — Il Salento ⭐

| Attribut | Wert |
|----------|------|
| Typ | Gastronomie |
| Koordinaten | 47.3793°N, 8.4949°E |
| Gesamtscore | 0.5054 |
| Distanz zu nächster Station | **3.12 km** |
| KDE-Score (norm.) | **1.0000** |

**Begründung:** Il Salento ist der besondere Fall unter den Top 5: Mit nur 3.12 km zur nächsten Ladestation hat dieser Standort den schlechtesten Distanz-Score — er liegt bereits in der Nähe bestehender Infrastruktur. Was ihn dennoch auf Rang 4 bringt: **Er liegt am frequentiertesten Velo-Hotspot des gesamten Kantons** (normierter KDE-Score = 1.0). Das Modell erkennt damit, dass manche Gebiete trotz bestehender Versorgung eine so hohe Nachfrage aufweisen, dass eine zusätzliche Station gerechtfertigt ist. Dies ist der einzige der Top-5-Standorte, der primär durch **Nachfrage** und nicht durch **Angebotslücke** legitimiert wird.

---

### Rang 5 — Gartentropfen

| Attribut | Wert |
|----------|------|
| Typ | Gastronomie |
| Koordinaten | 47.6316°N, 8.7977°E |
| Gesamtscore | 0.5000 |
| Distanz zu nächster Station | **14.04 km** |
| KDE-Score (norm.) | 0.0000 |

**Begründung:** Gartentropfen schliesst die geografische Abdeckung im **Nordwesten des Kantons**, einem Bereich, der durch die ersten vier Selektionen noch nicht abgedeckt war. Der KDE-Score von 0.0 weist zwar auf eine zählstellenschwache Zone hin, die 14 km Distanz zur nächsten Station zeigt aber eine klare Versorgungslücke. Für Velotouren in Richtung Zürcher Weinland / Thurtal ist dieser Standort strategisch wertvoll.

---

## Räumliche Verteilung

Die 5 gewählten Standorte decken den Kanton in alle Himmelsrichtungen ab:

```
Nordwest: Gartentropfen  (47.63°N)
Nordost:  NeftiHuus      (47.53°N)
Ost:      Freihof        (47.52°N / 8.89°E — östlichster)
Südost:   Hopp de Bäse!  (47.41°N)
Mitte/Süd: Il Salento    (47.38°N)
```

**Minimale Abstände zwischen gewählten Standorten:**
- Freihof ↔ NeftiHuus: ~24.8 km
- NeftiHuus ↔ Hopp de Bäse!: ~15.0 km  
- Hopp de Bäse! ↔ Il Salento: ~25.8 km
- Il Salento ↔ Gartentropfen: ~43.8 km

Durch den Greedy-Algorithmus mit 3-km-Ausschlusszone wird sichergestellt, dass keine zwei Standorte in unmittelbarer Nähe liegen — jede Station erschliesst ein eigenes Einzugsgebiet.

---

## Warum keine Bahnhöfe?

Obwohl 122 Bahnhöfe im Kandidatenpool waren, erscheint **kein einziger Bahnhof** unter den Top 5. Dafür gibt es zwei Gründe:

1. **Bestehende Versorgung:** Die grossen, frequentierten Bahnhöfe (Zürich HB, Winterthur, Uster usw.) liegen bereits in der 3-km-Versorgungszone bestehender Ladestationen und wurden beim Vorfilter ausgeschlossen.

2. **Schwächere KDE-Scores in Randlagen:** Kleinere Bahnhöfe in ländlichen Gebieten haben zwar grosse Distanzen zur nächsten Ladestation, aber auch kaum Velozählstellen in der Nähe — ihr KDE-Score bleibt niedrig. Sie konkurrieren damit mit Gastronomie-Standorten in denselben Gebieten, die aufgrund der Datenlage ähnlich abschneiden, aber als Rastpunkt-Infrastruktur besser geeignet sind.

---

## Limitationen

| Einschränkung | Auswirkung |
|---------------|------------|
| Gewichtung 50/50 ist eine Annahme | Andere Gewichte würden andere Top-5 erzeugen; eine Sensitivitätsanalyse wäre sinnvoll |
| OSM-Ladestationsdaten ggf. unvollständig | Möglicherweise mehr bestehende Stationen als die 11 im Modell |
| Kandidatenpool: nur Gastro + Bahnhöfe | Wanderparkplätze, Aussichtspunkte, Sportanlagen fehlen als POI-Typen |
| Euklidische Distanz | Nicht entlang des Velonetzes berechnet — echte Fahrdistanzen wären präziser |
| KDE basiert auf Zählstellennetz | In ländlichen Gebieten ohne nahe Zählstellen wird Frequenz systematisch unterschätzt |

---

## Datengrundlage

| Datensatz | Quelle | Umfang |
|-----------|--------|--------|
| Velozählstellen kantonsweit | Kanton ZH OGD (Dataset 3062) | 97 Stellen, DTV 2024–2025 |
| Velozählstellen Stadt ZH | Open Data Stadt Zürich | 24 Stellen mit Daten |
| Gastronomie-POIs | OpenStreetMap via OSMnx | 1'928 Standorte |
| Bahnhöfe | OpenStreetMap via OSMnx | 122 Standorte |
| Bestehende Ladestationen | OpenStreetMap (amenity=charging_station) | 11 Stationen |

---

*Erstellt im Rahmen des ZHAW-Semesterprojekts «Charge & Ride Züri», FS 2026.*
