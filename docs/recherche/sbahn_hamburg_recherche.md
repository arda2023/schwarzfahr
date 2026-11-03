# S-Bahn Hamburg – Systemrecherche für die App

Stand: 3. November 2026. Fahrplanzahlen stammen aus dem offiziellen hvv-GTFS-Datensatz vom 02.10.2026 (gültig bis 12.12.2026), ausgewertet für **Di 10.11.2026, Sa 14.11.2026 und So 15.11.2026**. Bauarbeiten können einzelne Tage verändern. Unsichere Punkte sind mit ⚠️ markiert.

Quellen des Datensatzes: [Transparenzportal Hamburg – hvv Fahrplandaten (GTFS) Okt.–Dez. 2026](https://suche.transparenz.hamburg.de/dataset?q=hvv+Fahrplandaten+GTFS), Lizenz dl-de/by-2.0 (Namensnennung „Hamburger Verkehrsverbund GmbH“ nötig).

---

## 1. Das Wichtigste in Kürze

- **Aktuell fahren 5 Linien: S1, S2, S3, S5 und S7.** Die S7 ist eine Übergangslinie. Seit Sommer 2025 ist die Brücke Holstenstraße–Altona nur in einer Richtung befahrbar. Deshalb fährt die S7 von Aumühle/Bergedorf durch den City-Tunnel nach Altona und über Dammtor zurück. Die S2 fährt nur als Verstärker in der Hauptverkehrszeit (HVZ) zwischen Bergedorf und Diebsteich. ([S-Bahn Hamburg: S7](https://www.s-bahn-hamburg.de/fahrplan/verkehrsmeldungen/s7))
- **⚠️ Ab Winter 2026/27** soll die S2 wieder Altona–Aumühle fahren, die S7 entfällt dann. Als Termin gilt wahrscheinlich der Fahrplanwechsel Mitte Dezember 2026, er ist aber nicht fest zugesagt. ([NDR, 21.09.2026](https://www.ndr.de/nachrichten/hamburg/s2-kehrt-bald-zurueck-nach-altona-s7-wird-eingestellt,s-bahn-174.html))
- **Grundtakt** ist tagsüber 10 Minuten, am frühen Morgen, spätabends und nachts 20 Minuten. S1 und S2 verdichten in der HVZ auf etwa 5 Minuten (genauer ein 4/6-Minuten-Takt). Die Außenäste Wedel, Aumühle, Buxtehude und Stade werden seltener bedient.
- **Flügelung gibt es nur bei der S1 in Ohlsdorf.** Bis Ohlsdorf fährt sie als Vollzug (2 Einheiten). Dort wird sie geteilt: Der **vordere** Zugteil fährt zum **Flughafen**, der **hintere** nach **Poppenbüttel**. In der Gegenrichtung werden die Teile in Ohlsdorf wieder vereinigt.
- **Gleichzeitig unterwegs** sind auf der S1 in der HVZ etwa **24 Züge bzw. Zugteile** (12 je Richtung), tagsüber etwa 12. Netzweit sind es in der HVZ etwa 80. Die Herleitung steht in Abschnitt 6.
- **Daten:** Der statische Fahrplan ist offen verfügbar (hvv-GTFS). Echtzeitdaten gibt es über die hvv-Geofox-API (GTI), aber nur auf Antrag. Die DB-Timetables-API ist kostenlos. Ein offener GTFS-RT-Feed liefert für Hamburg derzeit **keine Fahrzeugpositionen**.
- **Recht:** In Deutschland gibt es kein ausdrückliches Verbot von Kontrolleur-Warn-Apps und kein bekanntes Urteil dazu. In Frankreich ist so etwas strafbar (Fall Akha, Schadenersatz 32.020 €). Das größte praktische Risiko ist, dass die App aus den App-Stores entfernt wird (FreiFahren-iOS ist im deutschen App Store nicht mehr verfügbar).

---

## 2. Liniennetz (Stand November 2026)

Die S-Bahn Hamburg GmbH (DB) betreibt das Netz. Das meiste läuft mit 1200-V-Gleichstrom über Stromschiene. Nur Neugraben–Buxtehude–Stade (S5) fährt im Mischbetrieb unter 15-kV-Oberleitung, dafür braucht es Zweisystemzüge. Die Kernstrecken sind:
- **City-Tunnel:** Altona – Königstraße – Reeperbahn – Landungsbrücken – Stadthausbrücke – Jungfernstieg – Hbf. Hier fahren S1, S3 und S7 (Richtung Altona).
- **Verbindungsbahn:** Diebsteich/Altona – Holstenstraße – Sternschanze – Dammtor – Hbf. Hier fahren S2, S5 und S7 (Richtung Hbf).
- **Harburger S-Bahn:** Hbf – Hammerbrook – Elbbrücken – Veddel – Wilhelmsburg – Harburg – Neugraben. Hier fahren S3 und S5.

Quellen: [Wikipedia: S-Bahn Hamburg](https://de.wikipedia.org/wiki/S-Bahn_Hamburg), hvv-GTFS.

### Neustrukturierung Dezember 2023 und später
- Am **10.12.2023** wurde das Netz von 6 Linien (S1, S11, S2, S21, S3, S31) auf 4 Linien umgestellt (S1, S2, S3, S5). S11 und die alte Verstärker-S2 entfielen. Die alte S21 heißt seitdem S2, die alte S31 heißt S5. Die S3 wurde auf Neugraben verkürzt, die S5 bis Stade verlängert. Dazu kam der 5-Minuten-Takt in der HVZ auf S1 und S2. Die S3 fährt werktags von etwa 5 bis 19 Uhr als Langzug. ([Wikipedia](https://de.wikipedia.org/wiki/S-Bahn_Hamburg#10._Dezember_2023:_Neustrukturierung_des_Liniennetzes))
- Am **14.12.2025** kam die Liniennummer S7 hinzu (siehe oben).
- **S4 (Altona – Hasselbrook – Rahlstedt – Ahrensburg – Bad Oldesloe):** Die Strecke ist im Bau, der Start ist aber **auf unbestimmte Zeit verschoben**. Ursprünglich war Dezember 2027 bis Rahlstedt geplant. Es gibt noch keinen neuen Zeitplan, und für einen Abschnitt fehlt noch der Planfeststellungsbeschluss. Ab Hasselbrook teilt sich die S4 die S1-Gleise. Während der Bauarbeiten droht der S1 eine Phase mit ganztägigem 10-Minuten-Takt nördlich von Hasselbrook und einem Pendelverkehr Ohlsdorf–Poppenbüttel. Einen Termin dafür gibt es noch nicht. ([NDR](https://www.ndr.de/nachrichten/hamburg/s-bahn-ausbau-nach-rahlstedt-verspaetet-auf-unbestimmte-zeit,sbahn-172.html), [MOPO](https://www.mopo.de/hamburg/s4-laesst-auf-sich-warten-und-jetzt-drohen-monatelange-einschraenkungen-fuer-fahrgaeste/5060309), [Abendblatt](https://www.abendblatt.de/hamburg/politik/article411459266/neue-verzoegerungen-bei-hamburgs-s4-projekt-aktuell-ohne-zeitplan-2.html))
- **S5 nach Kaltenkirchen** über die bisherige AKN-Strecke ab Eidelstedt ist im Bau. Avisiert ist Ende 2028. Danach soll die S5 nicht mehr in Elbgaustraße enden, sondern nach Kaltenkirchen fahren. ([Wikipedia](https://de.wikipedia.org/wiki/S-Bahn_Hamburg#S5:_Kaltenkirchen_über_Eidelstedt))
- **S6 (Elbgaustraße – Verbindungsbahn – Neugraben)** ist geplant und für Ende 2029 avisiert. Später soll sie einen Ast zur Science City Bahrenfeld bekommen. Heute gibt es **keine S6**.

---

## 3. Linien im Detail (Stationsfolge mit Fahrzeit ab Startbahnhof)

Die Minutenangaben stammen aus dem hvv-GTFS (Di 10.11.2026, typische Fahrt).

### S1 – Wedel ⇄ Poppenbüttel / Hamburg Airport
| Abschnitt | Stationen (+Minuten ab Wedel) |
|---|---|
| Stamm | Wedel +0 · Rissen +4 · Sülldorf +7 · Iserbrook +10 · **Blankenese +13** · Hochkamp +16 · Klein Flottbek +18 · Othmarschen +21 · Bahrenfeld +23 · Ottensen +25 · **Altona +27** · Königstraße +29 · Reeperbahn +31 · Landungsbrücken +33 · Stadthausbrücke +36 · Jungfernstieg +37 · **Hauptbahnhof +40** · Berliner Tor +43 · Landwehr +46 · Hasselbrook +48 · Wandsbeker Chaussee +49 · Friedrichsberg +51 · Barmbek +54 · Alte Wöhr +56 · Rübenkamp +58 · **Ohlsdorf +60** |
| Ast Flughafen (vorderer Zugteil) | Hamburg Airport +66 |
| Ast Poppenbüttel (hinterer Zugteil) | Kornweg +66 · Hoheneichen +68 · Wellingsbüttel +70 · Poppenbüttel +72 |

Gesamtfahrzeit: 66 min nach Airport, 72 min nach Poppenbüttel (36 bzw. 30 Stationen). Die HVZ-Verstärker fahren Blankenese–Poppenbüttel in 56–58 min, ohne Flügelung.

### S2 – Bergedorf ⇄ Diebsteich (nur HVZ-Verstärker, Mo–Fr)
Diebsteich +0 · Holstenstraße +3 · Sternschanze +5 · Dammtor +8 · Hbf +11 · Berliner Tor +15 · Rothenburgsort +19 · Tiefstack +21 · Billwerder-Moorfleet +24 · Mittlerer Landweg +27 · Allermöhe +30 · Nettelnburg +32 · Bergedorf +34.
In den Wochenendnächten fährt die S2 zwischen Altona und Aumühle über die Verbindungsbahn, alle 20 Minuten.

### S7 – Aumühle/Bergedorf → City-Tunnel → Altona → Verbindungsbahn → Aumühle (Übergangslinie)
- Richtung Altona (44 min): Aumühle +0 · Wohltorf +3 · Reinbek +5 · Bergedorf +9 · Nettelnburg +12 · Allermöhe +14 · Mittlerer Landweg +16 · Billwerder-Moorfleet +20 · Tiefstack +22 · Rothenburgsort +24 · Berliner Tor +27 · Hbf +31 · Jungfernstieg +34 · Stadthausbrücke +36 · Landungsbrücken +38 · Reeperbahn +40 · Königstraße +42 · Altona +44
- Richtung Aumühle (44 min): Altona +0 · Holstenstraße +4 · Sternschanze +6 · Dammtor +8 · Hbf +11 · Berliner Tor +14 · … · Bergedorf +33 · Reinbek +37 · Wohltorf +40 · Aumühle +44
- **App-relevant:** Der Hin- und der Rückweg laufen über verschiedene Stationen. Ein Linienstreifen pro Richtung reicht hier nicht.

### S3 – Pinneberg ⇄ Neugraben (64–65 min)
Pinneberg +0 · Thesdorf +2 · Halstenbek +4 · Krupunder +7 · Elbgaustraße +10 · Eidelstedt +13 · Stellingen +15 · Langenfelde +17 · Diebsteich +19 · **Altona +23** · Königstraße +25 · Reeperbahn +27 · Landungsbrücken +29 · Stadthausbrücke +32 · Jungfernstieg +33 · **Hbf +36** · Hammerbrook +39 · Elbbrücken +42 · Veddel +44 · Wilhelmsburg +47 · **Harburg +52** · Harburg Rathaus +54 · Heimfeld +56 · Neuwiedenthal +62 · Neugraben +64.
In den Wochenendnächten fährt die S3 bis Stade.

### S5 – Elbgaustraße ⇄ Neugraben ⇄ Buxtehude/Stade
- Elbgaustraße +0 · Eidelstedt +2 · Stellingen +3 · Langenfelde +6 · Diebsteich +8 · Holstenstraße +11 · Sternschanze +13 · Dammtor +16 · **Hbf +18** · Hammerbrook +21 · Elbbrücken +24 · Veddel +27 · Wilhelmsburg +30 · **Harburg +35** · Harburg Rathaus +37 · Heimfeld +39 · Neuwiedenthal +44 · **Neugraben +47** · Fischbek +50 · Neu Wulmstorf +53 · Buxtehude +59 · Neukloster +63 · Horneburg +68 · Dollern +72 · Agathenburg +75 · **Stade +80**
- Elbgaustraße–Stade dauert etwa 80–82 min. Die Fahrten enden abwechselnd in Neugraben, Buxtehude oder Stade. Spät abends fährt die S5 nur Elbgaustraße–Berliner Tor. In den Wochenendnächten fährt sie gar nicht.
- **Datenfalle:** Im hvv-GTFS ist jede durchgehende S5 in **zwei Fahrten** aufgeteilt, nämlich Elbgaustraße–Neugraben und Neugraben–Buxtehude/Stade. Am 10.11. ließen sich 86 von 90 Fahrten stadtauswärts und 90 von 90 stadteinwärts zeitlich zu einem durchgehenden Zug zusammenfügen. In Neugraben wechselt der Zug von Gleichstrom auf Oberleitung.

---

## 4. Takt nach Tageszeit (Abfahrten pro Stunde und Richtung)

Gezählt sind Abfahrten pro Stunde und Richtung aus dem hvv-GTFS. 6/h entspricht einem 10-Minuten-Takt, 3/h einem 20-Minuten-Takt, 12/h etwa einem 5-Minuten-Takt.

| Linie / Messpunkt | Mo–Fr früh (5–6) | Mo–Fr HVZ (7–8, 15–18) | Mo–Fr tagsüber (9–14) | Abend (20–23) | Sa/So tagsüber | Wochenendnacht (ca. 1–4 Uhr) |
|---|---|---|---|---|---|---|
| S1 ab Wedel | 4–6 | 6 (10′) | 3 (20′) | 3 | 3 (20′) | 1 (60′) |
| S1 ab Blankenese | 5–6 | 10–12 (≈5′) | 6 | 6 | 6 | 3 (20′, bis Poppenbüttel) |
| S1 Ohlsdorf → Airport | 3–6 | 6 | 6 | 5–6 | 6 | 0 (nachts kein Airport) |
| S1 Ohlsdorf → Poppenbüttel | 3–5 | 12 (≈5′) | 6 | 6 | 6 | 3 |
| S7 ab Bergedorf | 5–6 | 6 | 6 | 6 → 3 | 6 | – (dann S2) |
| S7 ab Aumühle | 4–6 | 6 | 3 | 3 | 3 | – |
| S2 ab Bergedorf (Verstärker) | – | 6 (nur 6–9, 15–19 Uhr) | – | – | – | 3 (Altona–Bergedorf), Aumühle stündlich |
| Bergedorf gesamt (S2+S7) | 5–10 | 12 | 6 | 6 | 6 | 3 |
| S3 ab Pinneberg | 5–6 | 6 | 6 | 6 → 3 | 6 bzw. 3 ⚠️ | 3 |
| S5 ab Elbgaustraße | 6 | 6 | 6 | 6 | 6 (Sa/So vormittags weniger ⚠️) | – |
| Neugraben → Fischbek (S5/S3) | 6 | 6 | 3 | 3 | 2 (30′) | 1 |
| S5 Buxtehude → Stade | 2–3 | 3 (20′) | 1–3 | 1 | 1 (60′) | 1 (als S3) |

⚠️ Am Sa 14.11.2026 fährt die S3 ab Pinneberg nach 15 Uhr nur alle 20 Minuten, und die S5 ist vormittags ausgedünnt. Das sieht nach Bauarbeiten an diesem Tag aus und ist nicht der Regelfahrplan. Laut Wikipedia gilt sonst: Wedel und Aumühle alle 10 Minuten nur in der werktäglichen HVZ, sonst alle 20 Minuten; Buxtehude außerhalb der HVZ alle 20 Minuten, am Wochenende alle 30; Stade in der HVZ alle 20 Minuten, sonst stündlich. Der Nachtverkehr läuft durchgehend in den Nächten auf Samstag, Sonntag und Feiertage.

### Gebündelte Takte auf den Stammstrecken (Abfahrten/h je Richtung, Di 10.11.2026)
| Abschnitt | Linien | HVZ | tagsüber | Wochenende tagsüber |
|---|---|---|---|---|
| City-Tunnel Hbf → Jungfernstieg → Altona | S1+S3+S7 | 24 (Ø 2,5 min) | 18 (Ø 3,3 min) | 18 |
| Verbindungsbahn Dammtor → Hbf | S2+S5+S7 | 18 | 12 | 12 |
| Verbindungsbahn Hbf → Dammtor | S2+S5 (S7 fährt hier nur Richtung Hbf) | 12 | 6 | 6 |
| Hbf → Harburg (Harburger S-Bahn) | S3+S5 | 12 | 12 | 12 (morgens weniger) |

Wikipedia nennt die kürzeste Zugfolge im Bestandsnetz mit etwa 3 Minuten. Mit ETCS sollen bis zu 90 Sekunden möglich werden.

---

## 5. Flügelung, Zuglängen und Fahrzeuge

### Zugbildung
| Bezeichnung | Einheiten | Wagen | Länge ca. |
|---|---|---|---|
| Kurzzug | 1 | 3 | 66 m |
| Vollzug | 2 | 6 | 132 m |
| Langzug | 3 | 9 | 198 m |

Planmäßige Langzüge fahren nur auf der **S3**, werktags von etwa 5 bis 19 Uhr. Alle anderen Linien fahren höchstens als Vollzug. ([Wikipedia](https://de.wikipedia.org/wiki/S-Bahn_Hamburg#Fahrzeuge))

### Wo wird geteilt oder vereinigt?
| Ort | Linie | Was passiert | Quelle / Status |
|---|---|---|---|
| **Ohlsdorf** | S1 | Der Vollzug von Wedel/Blankenese wird geteilt. Der **vordere** Teil fährt zum Flughafen, der **hintere** nach Poppenbüttel. In der Gegenrichtung werden beide Teile in Ohlsdorf vereinigt. | Wikipedia; bestätigt durch GTFS. Am 10.11. gab es 82 identische Fahrtpaare je Richtung von Wedel und 25 bis 28 von Blankenese. |
| Ohlsdorf (HVZ-Verstärker) | S1 | Die Verstärker Blankenese ⇄ Poppenbüttel fahren **ohne** Flughafen-Teil (35 bzw. 21 Fahrten am 10.11.) | GTFS |
| Neugraben | S5 | **Keine Flügelung.** Der Zug fährt durch und wechselt dabei das Stromsystem. Nur im GTFS ist er in zwei Fahrten geteilt. | GTFS-Analyse ⚠️ (Schwächen oder Stärken in Neugraben selbst nicht verifiziert) |
| Endbahnhöfe allgemein | alle | Verstärken bzw. Schwächen (Kurzzug ↔ Vollzug) je nach Tageszeit ist üblich. Früher wurde z. B. die S1 in Ohlsdorf in der Schwachverkehrszeit geschwächt. | Wikipedia ⚠️ (konkrete Umläufe nicht öffentlich) |

### Fahrzeuge
| Baureihe | Anzahl | Sitz-/Stehplätze je Einheit (3 Wagen) | Höchstgeschwindigkeit | Einsatz | Besonderheit |
|---|---|---|---|---|---|
| **474 / 474plus** (Adtranz/LHB, 1996–2001, 2006) | 112 Einheiten (70 Einsystem, 42 Zweisystem 474.3) | 208 / 306 | 100 km/h | S1, S2, S5, S7 | Scharfenbergkupplung, Wagenübergänge seit dem Umbau zu „474plus“. 4046/4047 fahren mit ATO/ETCS auf der Bergedorfer Strecke. |
| **490** (Bombardier/Alstom, 2013–2020, weitere 2025–2026) | 82 ausgeliefert (51 Einsystem, 31 Zweisystem), 64 weitere bestellt | 172 + 18 Klappsitze / 279 | 100 km/h, Zweisystem 140 km/h | S2, S3, S5 | durchgehend begehbar, Klimaanlage, Scharfenbergkupplung |

Die Baureihen 474 und 490 lassen sich **nicht miteinander kuppeln**. Ein Zug besteht also immer aus 1 bis 3 Einheiten derselben Baureihe. Nach Stade fahren nur Zweisystem-Einheiten (474.3 oder 490.2). Ein Vollzug fasst rund 1.000 Fahrgäste, ein Langzug rund 1.400 bis 1.500. Quellen: [Wikipedia BR 474](https://de.wikipedia.org/wiki/DB-Baureihe_474), [Wikipedia BR 490](https://de.wikipedia.org/wiki/DB-Baureihe_490), [Wikipedia S-Bahn Hamburg](https://de.wikipedia.org/wiki/S-Bahn_Hamburg#Fahrzeuge).

---

## 6. Wie viele Züge sind gleichzeitig unterwegs?

### Methode
Gemessen habe ich mit dem hvv-GTFS. Für einen Zeitpunkt *t* werden alle Fahrten gezählt, die sich gerade zwischen zwei Halten befinden oder an einem Halt stehen. Bei geflügelten S1-Paaren, die bis Ohlsdorf identisch fahren, zählt der Zug bis Ohlsdorf **einmal** und danach **zweimal** (Airport-Teil und Poppenbüttel-Teil). Nicht mitgezählt sind Züge, die an Endstationen wenden oder abgestellt sind. Die Zahl der eingesetzten Fahrzeuge liegt deshalb höher.

| Zeitpunkt | S1 | S2 | S7 | S3 | S5 | Summe |
|---|---|---|---|---|---|---|
| Di 07:45 (HVZ) | 27 (13 / 14 je Richtung) | 6 | 9 | 12 | 15 | 69 |
| Di 08:30 (HVZ) | 24 (12 / 12) | 7 | 9 | 14 | 12 | 66 |
| Di 12:00 | 12 (6 / 6) | 0 | 7 | 14 | 10 | 43 |
| Di 17:00 (HVZ) | 24 (12 / 12) | 7 | 9 | 14 | 12 | 66 |
| Di 21:00 | 12 | 0 | 7 | 14 | 10 | 43 |
| Sa 14:00 | 12 | 0 | 7 | 14 | 10 | 43 |
| So 11:00 | 12 | 0 | 7 | 12 | 10 | 41 |
| Nacht Sa→So 02:00 | 6 | 4 | 0 | 9 | 0 | 19 |

(Werte für S5 gelten ab Elbgaustraße einschließlich der Außenäste. Bei der S3 sind Langzüge als 1 Zug gezählt.)

### Plausibilitätsrechnung (Faustformel)
Die Faustformel lautet: Züge in Fahrt je Richtung ≈ Fahrzeit ÷ Takt. Für den Fahrzeugbedarf rechnet man (2 × Fahrzeit + 2 × Wendezeit) ÷ Takt.

- **S1 tagsüber** (Mo–Fr 9–14 Uhr): Blankenese–Ohlsdorf dauert 46 min im 10-Minuten-Takt, das ergibt etwa 4,6 Züge je Richtung. Dazu kommen kurze Flügelstrecken (Airport 6 min, Poppenbüttel 12 min) und der 20-Minuten-Ast Wedel (13 min ÷ 20). Das ergibt etwa 6 je Richtung und passt zu den gemessenen **6 / 6**.
- **S1 HVZ:** Wedel–Poppenbüttel dauert 72 min. Der Stamm läuft im 5-Minuten-Takt. 46 min ÷ 5 ergibt etwa 9 Züge, dazu kommen die Äste, insgesamt etwa 12 je Richtung. Das passt zu den gemessenen **12 / 12**. Mit Wendezeiten ergibt sich ein Fahrzeugbedarf von grob 30 Zügen bzw. 50 bis 60 Einheiten (Vollzüge!) ⚠️ (Schätzung, Umlaufpläne sind nicht öffentlich).
- **S3:** 64,5 min Fahrzeit im 10-Minuten-Takt ergibt etwa 6,5 Züge je Richtung, insgesamt **13 bis 14**. Gemessen: 14.

**Für die App heißt das:** Auf der S1 gibt es pro Richtung tagsüber etwa 6 und in der HVZ etwa 12 „Karten“ zum Wischen. Hinzu kommen die Flügelteile nach Ohlsdorf.

---

## 7. Daten und APIs für die App

| Quelle | Inhalt | Zugang / Lizenz | Fahrzeugpositionen? | Bewertung |
|---|---|---|---|---|
| **hvv-GTFS (Transparenzportal)** | Sollfahrplan aller hvv-Linien inkl. S-Bahn, Haltestellen (DHID-IDs, z. B. `de:02000:…`), Shapes | frei, monatliche Updates, dl-de/by-2.0, Namensnennung nötig ([Portal](https://suche.transparenz.hamburg.de/dataset?q=hvv+Fahrplandaten+GTFS), [hvv Datenabruf](https://www.hvv.de/de/fahrplaene/abruf-fahrplaninfos/datenabruf)) | nein | Basis für Linienstreifen und Soll-Positionen. Es gibt **keine Zugnummern** (`trip_short_name` ist leer) und **keine block_id**. Flügelteile sind 2 parallele Fahrten, die S5 ist in Neugraben geteilt. |
| **Geofox GTI (hvv/HOCHBAHN)** | departureList (Echtzeit-Abfahrten mit Verspätung), Fahrtverlauf, Verbindungsauskunft, Haltestellen- und Linienlisten, `getVehicleMap` (Fahrzeuge in einer Bounding Box) | **nur auf Antrag**, kein Anspruch. Kostenlos, aber laut Nutzungsbedingungen zweckgebunden für eine „kostenfreie Fahrplanauskunft für Fahrgäste“. Requests werden mit HMAC-SHA1 signiert. ([hvv Datenabruf](https://www.hvv.de/de/fahrplaene/abruf-fahrplaninfos/datenabruf), [GTI-Handbuch](https://gti.geofox.de/html/GTIHandbuch_p.html)) | ja, über `getVehicleMap`, ⚠️ vermutlich aus Fahrplan und Verspätung interpoliert und kein GPS | Fachlich die beste Quelle. **⚠️ Für eine Kontrolleur-Warn-App ist eine Freigabe sehr unwahrscheinlich**, weil der hvv selbst Auftraggeber der Prüfdienste ist. |
| **DB Timetables API** (DB API Marketplace) | Abfahrtstafeln pro DB-Station (EVA-Nr.) mit Zugnummern, Gleis und Echtzeitänderungen (`/plan`, `/fchg`, `/rchg`) | kostenlos, 60 Aufrufe/min, CC BY 4.0 ([Marketplace](https://developers.deutschebahn.com/db-api-marketplace/apis/product/timetables)) | nein | Gute Quelle für **Zugnummern** und Verspätungen der S-Bahn an DB-Stationen. Das Limit ist knapp (rund 70 Stationen); man braucht einen Server-Cache. |
| **DB RIS::Journeys** | Fahrtverläufe inkl. S-Bahn Hamburg, **Vereinigung/Durchbindung** (also Flügelung) | kostenpflichtig, nur nach Prüfung durch die DB, per Vertrag ([Marketplace](https://developers.deutschebahn.com/db-api-marketplace/apis/product/ris-journeys-transporteure)) | nein | Ideal für Flügelzüge, aber für eine solche App realistisch kaum zu bekommen. |
| **gtfs.de GTFS-RT** | TripUpdates und ServiceAlerts, deutschlandweit, alle 10 s | frei, CC BY-SA 4.0 ([gtfs.de/realtime](https://gtfs.de/de/realtime/)) | **nein** (keine VehiclePositions) | Hamburg fehlt weitgehend: „Hamburger Hochbahn – in Vorbereitung / Klärung von Lizenzfragen“. Ob die DB-S-Bahn Hamburg über „SIRI Bahn (Regionalverkehr)“ oder DELFI enthalten ist, ⚠️ ist unklar und müsste man testen. Der Feed passt zum gtfs.de-Statik-Feed, nicht zum hvv-GTFS. |
| **DELFI** (bundesweiter Soll- und Echtzeitdatensatz) | Sollfahrplan Deutschland plus DELFI-Realtime (Prognosen) als GTFS-RT | Statik offen; zur Echtzeit siehe [delfi.de](https://www.delfi.de/) ⚠️ (Bedingungen nicht geprüft) | nein | Erst bei einer bundesweiten Skalierung interessant |
| **Transitous** (Community, MOTIS) | Abfahrten mit Echtzeit, deutschlandweit, ohne Schlüssel | Fair Use ([Doku](https://docs.openpublictransport.net/providers/db/)) | nein | gut für Prototypen, nicht für Produktion |

### Wie ordnet man einen Nutzer einem konkreten Zug zu?
1. **Ein „Zug“ ist für die App eine Fahrt.** Ein Zug lässt sich über Linie, Richtung, Abfahrtszeit am Startbahnhof und Datum eindeutig benennen. Eine DB-Zugnummer aus der Timetables-API kann man als zusätzliche Kennung anhängen. ⚠️ Das Nummernschema der S-Bahn Hamburg habe ich nicht verifiziert.
2. **Position ableiten:** Aus Sollzeit plus Verspätung (GTI departureList oder DB Timetables) lässt sich berechnen, zwischen welchen zwei Stationen jede Fahrt gerade ist. Echte GPS-Positionen gibt es offen nicht.
3. **Nutzer matchen:** Der Nutzer tippt beim Einsteigen auf „Ich bin in diesem Zug“, die Station ergibt sich aus dem GPS. Danach plausibilisiert die App laufend: Bewegt sich der GPS-Punkt entlang der Strecke (Shapes aus dem GTFS) synchron zur errechneten Fahrtposition? Wenn sich der Nutzer an einer Station nicht mehr bewegt und der Zug abfährt, wurde er „ausgestiegen“. Im City-Tunnel (Altona–Hbf) und im Harburger Tunnel gibt es kaum GPS. Dort hilft nur die Fahrplanlogik oder Funkzellen bzw. WLAN.
4. **Flügelzüge:** Bis Ohlsdorf ist das Paar Airport/Poppenbüttel **ein physischer Zug, aber zwei Fahrten**. Eine Meldung „Kontrolleure im Zug“ vor Ohlsdorf muss beiden Fahrten zugeordnet werden. Fragt die App den Wagen ab („vorderer oder hinterer Zugteil?“), kann sie die Meldung ab Ohlsdorf der richtigen Hälfte zuordnen. Vorne heißt Airport, hinten heißt Poppenbüttel.

---

## 8. Fahrkartenkontrollen in Hamburg (faktisch)

- **Wer kontrolliert?** In der S-Bahn kontrolliert der **Prüfdienst der S-Bahn Hamburg**, laut eigenem Magazin rund 30 Mitarbeitende, an 7 Tagen pro Woche, meist in Teams. In U-Bahn und Bus kontrolliert die Hochbahn-Wache. Zusätzlich gibt es seit 2026 **hvv-Prüfteams von vhh.mobility** in den Umlandkreisen, z. B. Pinneberg, Stade und Stormarn, auch auf S-Bahn-Linien. Dazu kommen gemeinsame **Großkontrollen** und angekündigte „Prüfmarathons“. ([S-Bahn-Magazin „Auf Streife“](https://www.s-bahn-hamburg.de/magazin/Betrieb/Auf-Streife-2), [hvv-PM Umland](https://www.hvv.de/resource/blob/171402/954b7431814f3df353a50bfccf07c391/PM260422_Pr%C3%BCfteams_Umland.pdf))
- **Arten:** **Streckenkontrolle** (im fahrenden Zug) und **Abgangskontrolle** (am Bahnsteig oder Ausgang). Laut hvv-Beförderungsbedingungen endet die Fahrt erst nach der Kontrolle, wenn beim Aussteigen erkennbar Prüfpersonal bereitsteht. Für die App heißt das: Kontrolleure stehen nicht nur „in einem Zug“, sondern oft **auf einem Bahnsteig**. Beide Meldungstypen sind nötig.
- **Umfang:** Die S-Bahn hat 2024 rund **3,8 Mio. Fahrgäste kontrolliert**, 2022 waren es 1,4 Mio. ([Bürgerschafts-Drucksache 23/507](https://www.buergerschaft-hh.de/parldok/dokument/91132/23_00507_wie_steht_es_um_das_schwarzfahren_in_hamburgs_bussen_und_bahnen_im_jahr_2025_ii)). Die Quote ohne gültiges Ticket lag laut Presse in den ersten neun Monaten 2025 bei 4,1 % im hvv ([WELT](https://www.welt.de/regionales/hamburg/article68f5cb329c193ceaa4c4dc12/ticketkontrollen-mehr-schwarzfahrer-in-hamburgs-bussen-und-bahnen-erwischt.html), ⚠️ Zahl nur aus dem Snippet).
- **Folgen:** Das erhöhte Beförderungsentgelt kostet **60 €**. Wer eine persönliche Zeitkarte vergessen hat und sie nachweist, zahlt 7 € (§ 9 hvv-Beförderungsbedingungen). Bei der S-Bahn wird über [db-fn.de](https://www.db-fn.de) abgewickelt. **Strafrechtlich** gilt § 265a StGB (Erschleichen von Leistungen): Geldstrafe oder Freiheitsstrafe bis 1 Jahr. ([hvv-Tarifbestimmungen](https://www.hvv.de/resource/blob/2384/e531b8525a594b140bc38f5369e1fa6c/hvv_tarif_gemeinschaftstarif_befoerderungsbedingungen.pdf), [S-Bahn Hamburg](https://www.s-bahn-hamburg.de/fahrplan/fahrkartenkontrolle))

---

## 9. Rechtslage von Kontrolleur-Warn-Apps (keine Rechtsberatung)

Die Konkurrenz-Analyse (Apps, Nutzerzahlen, Funktionen) steht in **`/workspace/konkurrenz_kontrolleur_apps.md`**. Hier folgen nur die rechtlich belegten Punkte.

| Punkt | Befund | Quelle |
|---|---|---|
| Ausdrückliches Verbot in DE? | **Nein.** § 23 Abs. 1c StVO verbietet nur Autofahrern Blitzerwarner. Für den ÖPNV gibt es keine vergleichbare Norm. | [TECHBOOK/Solmecke](https://www.techbook.de/mobile-lifestyle/apps/warn-app-bvg), [presse.online](https://presse.online/2026/08/15/40-000-nutzer-bvg-kritisiert-freifahren-app/) |
| Urteil in DE? | **Kein bekanntes Urteil** zu einer ÖPNV-Kontrollwarn-App (Stand Aug. 2026). Die BVG prüft laut Presse rechtliche Schritte. | [iphone-ticker](https://www.iphone-ticker.de/freifahren-berliner-app-warnt-vor-ticketkontrollen-283554/), [presse.online](https://presse.online/2026/08/15/40-000-nutzer-bvg-kritisiert-freifahren-app/) |
| Strafbarkeit des Betreibers? | Wahre Ortsangaben sind laut Medienanwalt Solmecke von der Meinungsfreiheit gedeckt, **solange nicht zum Fahren ohne Ticket aufgefordert wird**. Diskutiert wird eine **Beihilfe** (§ 27 StGB i. V. m. § 265a StGB). Ob sie greift, ist offen und hängt von der Ausgestaltung ab. Darüber hinaus diskutieren Juristen zivilrechtliche Unterlassungsansprüche der Verkehrsunternehmen. ⚠️ (ungeklärt) | [BILD/Solmecke](https://www.bild.de/leben-wissen/digital/wie-legal-ist-die-bvg-warn-app-freifahren-star-anwalt-ordnet-ein-6a7b27ec0e3689919ec77f6f), [presse.online](https://presse.online/2026/08/15/40-000-nutzer-bvg-kritisiert-freifahren-app/) |
| Datenschutz | Ergeben sich aus den Meldungen **Bewegungsprofile einzelner Kontrolleure** (oder Fotos und Beschreibungen), greift die DSGVO. Anonyme Orts- und Zeitangaben gelten als deutlich weniger problematisch. | [Verbraucherschutzforum](https://verbraucherschutzforum.berlin/2026-08-13/kontrollen-in-echtzeit-warum-die-warn-app-freifahren-rechtlich-heikel-bleibt-430238/), BILD |
| Frankreich (Präzedenzfall) | **Verboten** nach Art. L2242-10 Code des transports (bis 2 Monate Haft und 3.750 €). Die App **Akha** (Paris) wurde im **Januar 2025 aus Google Play und dem Apple App Store entfernt**. Am **7.5.2025** wurde der Gründer verurteilt, **32.020 €** Schadenersatz zu zahlen (20.000 € an IDFM, 5.720 € an RATP, 6.300 € an SNCF), vorläufig vollstreckbar. **✔ verifiziert.** | [Le Figaro (Store)](https://www.lefigaro.fr/secteur/high-tech/valerie-pecresse-veut-faire-fermer-akha-une-application-qui-signale-la-presence-de-controleurs-ratp-20250113), [Le Figaro (Urteil)](https://www.lefigaro.fr/secteur/high-tech/le-createur-de-l-application-akha-qui-signalait-la-presence-de-controleurs-ratp-et-sncf-condamne-au-civil-20250513), [actu.fr](https://actu.fr/ile-de-france/paris_75056/son-application-permettait-d-echapper-aux-controleurs-sncf-et-ratp-le-createur-d-akha-lourdement-condamne_62635280.html) |
| FreiFahren (Berlin), iOS-App | Am 03.11.2026 liefert die App-Store-Seite (id6738277309, DE) **404**, und die iTunes-Lookup-API findet 0 Treffer. Die App ist also **nicht mehr im deutschen App Store**. Einen Leserhinweis auf die Entfernung gab es am 22.08.2026. **⚠️ Den Grund (Apple oder Entwickler) hat niemand offiziell bestätigt.** Die Web-App läuft weiter. | eigene Prüfung, [iphone-ticker](https://www.iphone-ticker.de/freifahren-berliner-app-warnt-vor-ticketkontrollen-283554/), [app.freifahren.org](https://app.freifahren.org/) |
| Hamburg | Es gibt schon die Telegram-Gruppe „Kontrollettis HH“ und die App „KontrollRadar“. Laut moin.de sind solche Warngruppen legal. ⚠️ (Pressebewertung, kein Urteil) | [moin.de](https://www.moin.de/norddeutschland/hvv-schwarzfahrer-gruppen-warnungen-facebook-organisieren-id300837690.html), Konkurrenz-Datei |

**Was daraus folgt:** Das rechtliche Risiko in Deutschland ist derzeit eher **zivilrechtlich und plattformbezogen** als strafrechtlich. Die Formulierungen sollten neutral bleiben (kein „Schwarzfahrer“-Branding, keine Aufforderung zum Fahren ohne Ticket), und es sollten keine personenbezogenen Daten über Kontrolleure gespeichert werden. Eine PWA bzw. Web-App mitzudenken, schützt gegen eine Entfernung aus den Stores. Vor dem Launch lohnt sich eine kurze anwaltliche Prüfung. Der Name „Schwarzfahrer-App“ ist rechtlich und für die App-Store-Freigabe die riskanteste Variante.

---

## 10. Was das für das App-Design bedeutet

1. **Das Wisch-Modell passt gut zur Realität.** Jede Linie und Richtung hat eine geordnete Liste aktiver Fahrten. Wischt der Nutzer nach links, sieht er die vorherige Fahrt, also den Zug, der vorher abgefahren ist. Auf der S1 sind das tagsüber etwa 6 und in der HVZ etwa 12 Karten je Richtung. Der Abstand beträgt 10 bzw. etwa 5 Minuten, auf dem Ast Wedel 20 Minuten.
2. **Zwei Ebenen von Meldungen:** (a) **Station/Bahnsteig** („Abgangskontrolle in Blankenese“), sie gilt für alle Züge dort. (b) **Fahrt** („Kontrolleure in S1 08:12 ab Wedel“), sie wandert mit dem Zug den Streifen entlang. Ein eigener Ereignistyp „Kontrolleure ausgestiegen in X“ verschiebt die Meldung von der Fahrt auf die Station. Das ist genau der Fall des Nutzers in Altona.
3. **Flügelzüge darstellen:** Der S1-Streifen gabelt sich nach Ohlsdorf (Airport vorne, Poppenbüttel hinten). Bis Ohlsdorf ist eine Karte ein Zug mit zwei Zielen. Danach entstehen zwei Karten. Eine Meldung im Zug wird an Ohlsdorf auf beide Teile vererbt, sofern der Melder nicht „vorderer/hinterer Teil“ angegeben hat. Nach dem Teilen wechseln Kontrolleure übrigens häufig nicht den Zugteil. Das ist eine plausible Annahme, ⚠️ aber nicht belegt.
4. **Besondere Linienformen:** Die S7 ist eine Schleife mit unterschiedlichen Stationen je Richtung. Die S5 endet abwechselnd in Neugraben, Buxtehude oder Stade, also braucht es wechselnde Endpunkte auf dem Streifen. Die S2 fährt nur in der HVZ. Ab Dezember 2026 ändert sich das Netz voraussichtlich wieder (S2 zurück nach Altona, S7 entfällt). Deshalb sollte man **Linien nie hart codieren**, sondern bei jedem Fahrplanwechsel aus dem GTFS generieren.
5. **Gemeinsame Abschnitte:** Im City-Tunnel fahren S1, S3 und S7, auf der Verbindungsbahn S2, S5 und S7, nach Harburg S3 und S5. Steigen Kontrolleure in Jungfernstieg aus, kann der nächste Zug eine andere Linie sein. Stationsmeldungen sollten deshalb **linienübergreifend** auf allen Linien dieser Station erscheinen.
6. **Echtzeit:** Mit reinen Sollzeiten aus dem GTFS funktioniert ein MVP. Für „wann kommt mein Zug“ braucht man Verspätungen, am besten über die DB Timetables API (kostenlos, Zugnummern, aber ein Limit von 60 Aufrufen/min) mit einem Server-Cache. Mit einer Freigabe für Geofox GTI sollte man bei diesem Zweck nicht rechnen.
7. **Aussteigen erkennen:** Wenn der Nutzer zustimmt, das GPS mit der Fahrtposition abgleichen. Steht der Nutzer still, während der Zug abfährt, ist er ausgestiegen. In den Tunneln gibt es kaum GPS. Deshalb sollte zusätzlich ein Button „Ich bin ausgestiegen“ angeboten werden. Standortdaten möglichst auf dem Gerät verarbeiten, so wie bei FreiFahren.
8. **Verfall und Vertrauen:** Fahrtmeldungen sollten verfallen, sobald die Fahrt endet. Stationsmeldungen sollten nach 20 bis 60 Minuten verblassen. Bestätigungen anderer Fahrgäste in derselben Fahrt erhöhen das Vertrauen in eine Meldung.

---

## 11. Quellen (Auswahl)
- Wikipedia: [S-Bahn Hamburg](https://de.wikipedia.org/wiki/S-Bahn_Hamburg), [DB-Baureihe 474](https://de.wikipedia.org/wiki/DB-Baureihe_474), [DB-Baureihe 490](https://de.wikipedia.org/wiki/DB-Baureihe_490)
- S-Bahn Hamburg: [S7-Info](https://www.s-bahn-hamburg.de/fahrplan/verkehrsmeldungen/s7), [Fahrkartenkontrolle](https://www.s-bahn-hamburg.de/fahrplan/fahrkartenkontrolle), [Magazin „Auf Streife“](https://www.s-bahn-hamburg.de/magazin/Betrieb/Auf-Streife-2)
- hvv: [Datenabruf / GTI-Bedingungen](https://www.hvv.de/de/fahrplaene/abruf-fahrplaninfos/datenabruf), [Tarif- und Beförderungsbedingungen](https://www.hvv.de/resource/blob/2384/e531b8525a594b140bc38f5369e1fa6c/hvv_tarif_gemeinschaftstarif_befoerderungsbedingungen.pdf), [GTFS im Transparenzportal](https://suche.transparenz.hamburg.de/dataset?q=hvv+Fahrplandaten+GTFS)
- [Geofox GTI](https://gti.geofox.de/), [DB Timetables API](https://developers.deutschebahn.com/db-api-marketplace/apis/product/timetables), [DB RIS::Journeys](https://developers.deutschebahn.com/db-api-marketplace/apis/product/ris-journeys-transporteure), [gtfs.de Realtime](https://gtfs.de/de/realtime/), [DELFI](https://www.delfi.de/)
- Presse: [NDR S2/S7](https://www.ndr.de/nachrichten/hamburg/s2-kehrt-bald-zurueck-nach-altona-s7-wird-eingestellt,s-bahn-174.html), [NDR S4](https://www.ndr.de/nachrichten/hamburg/s-bahn-ausbau-nach-rahlstedt-verspaetet-auf-unbestimmte-zeit,sbahn-172.html), [MOPO S4/S1](https://www.mopo.de/hamburg/s4-laesst-auf-sich-warten-und-jetzt-drohen-monatelange-einschraenkungen-fuer-fahrgaeste/5060309), [Bürgerschaft Drs. 23/507](https://www.buergerschaft-hh.de/parldok/dokument/91132/23_00507_wie_steht_es_um_das_schwarzfahren_in_hamburgs_bussen_und_bahnen_im_jahr_2025_ii), [Le Figaro Akha-Urteil](https://www.lefigaro.fr/secteur/high-tech/le-createur-de-l-application-akha-qui-signalait-la-presence-de-controleurs-ratp-et-sncf-condamne-au-civil-20250513)
- Eigene Auswertung: Skripte und Daten in `/workspace/sbhh/` (hvv-GTFS vom 02.10.2026)
