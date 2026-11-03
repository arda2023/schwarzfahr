# Echtzeitdaten für die S-Bahn Hamburg: Quellen, Abfrageintervalle, Kosten

Stand: 03.11.2026. Bezug: KONZEPT.md Abschnitte 2 und 6.2–6.4, sbahn_hamburg_recherche.md Abschnitt 7.

> **Methodik und Grenzen:** In dieser Sitzung waren die meisten Websites nicht direkt abrufbar (hvv.de, developers.deutschebahn.com, gtfs.de, supabase.com, mobilithek.info waren gesperrt). Belegt wurde deshalb über Websuche (Ausschnitte der Originalseiten), über GitHub-Quelltexte und über den Datensatz `db-hafas-stations` (npm). Alles, was ich nur aus Zweitquellen kenne oder gar nicht prüfen konnte, ist mit ⚠️ markiert. Vor einer Entscheidung sollte man die verlinkten Originalseiten einmal selbst öffnen.

---

## Kurzfazit

1. **Grok fragt 1–3 Mal am Tag die hvv-App ab: das bringt für die Zugzuordnung nichts.** Verspätungen ändern sich minütlich. Brauchbar wäre ein Abruf jeder Stützstation alle 30–60 s. Das sind über 100.000 Abrufe am Tag. Dafür ist ein LLM das falsche Werkzeug (teuer, langsam, kann Zahlen erfinden, bedient keine App). Rechtlich ist es außerdem riskant (Zweckbindung der hvv-Daten, Datenbankschutz nach § 87b UrhG).
2. **Die beste offizielle Quelle bleibt die DB Timetables API.** Sie ist kostenlos, hat 60 Aufrufe/min, steht unter CC BY 4.0 ⚠️ (nur über Zweitquelle belegt) und enthält S-Bahn-Hamburg-Züge mit Echtzeit. Alle rund 66 S-Bahn-Stationen haben eine EVA-Nummer (Liste unten).
3. **Mit 60 Aufrufen/min** kommt man auf etwa 33–38 s pro Station für die ganze S1 und etwa 75 s pro Station fürs ganze Netz. Klüger ist es, nur 12–15 **Stützstationen** alle 15–20 s abzufragen. Das reicht für den 5-Minuten-Takt sicher. Im 2,5-Minuten-Takt im City-Tunnel reicht es nur zusammen mit der Reihenfolge-Logik und der Frage „Was steht am Zug?“.
4. **Als zweite Quelle testen:** den DELFI-Echtzeitfeed (GTFS-RT/SIRI-ET über Mobilithek bzw. gtfs.de). Ob die S-Bahn Hamburg darin vollständig enthalten ist, ⚠️ ist offen. Die hvv/Hochbahn bieten weiterhin **keine** offenen Echtzeitdaten an. v6.db.transport.rest ist **abgeschaltet**. Transitous ist nicht für kommerzielle Nutzung gedacht.
5. **Kosten:** Die Rechnung des Reviewers (28,8 Mio. Nachrichten) ist arithmetisch richtig. Bei Supabase Pro kostet das rund **95 $/Monat** (25 $ Basis + 60 $ Nachrichten + 10 $ Verbindungen). Sendet man nur Änderungen, kommt man auf etwa **35 $**. Ein CDN-Snapshot, den die App alle 15–30 s abfragt, kostet etwa **25–35 $** inklusive Supabase-Basis. Fazit: Verspätungen als Snapshot ausliefern, nur Meldungen per Push.

---

## 1. „Grok-Abruf der hvv-App“: Bewertung

### 1.1 Was bringt ein Abruf 1–3 Mal am Tag?

**Für Verspätungen und die Zugzuordnung bringt er nichts.**

- Die Zuordnung (KONZEPT 6.2/6.3) braucht die Verspätung *jetzt*. Liegen zwei Kandidaten unter 3 min auseinander, muss die App unterscheiden können. Im City-Tunnel fährt alle 2,5 min eine S-Bahn. Die Schätzung muss dort also besser als etwa ±1 min sein.
- Eine Verspätung von 08:00 Uhr sagt um 08:20 Uhr fast nichts mehr. S-Bahn-Verspätungen bauen sich innerhalb weniger Halte auf oder ab. Bei Störungen springen sie in Minuten von 0 auf 10+ min.
- Ein Abruf 3 Mal täglich an „einigen Stationen“ liefert also 3 veraltete Momentaufnahmen. Höchstens für eine Statistik („die S3 ist morgens im Schnitt 2 min zu spät“) taugt das etwas. Dafür gibt es aber bessere Wege (eigene Crowd-Durchfahrten, DB-Daten).

### 1.2 Wie oft müsste man abfragen, damit es etwas bringt?

| Ziel | Nötiges Alter der Verspätung | Abfrage je Station | Stationen | Abrufe |
|---|---|---|---|---|
| 5-Min-Takt S1 sicher trennen (Fehler < ±2 min) | ≤ 1–2 min | alle 60 s | ~12 Stützstationen | ~12/min ≈ 17.000/Tag |
| 2,5-Min-Takt City-Tunnel (Fehler < ±1 min) | ≤ 30–60 s | alle 30 s | Tunnelstationen + Zuläufe, ~15 | ~30/min ≈ 43.000/Tag |
| ganzes Netz, jede Station | ≤ 60 s | alle 30–60 s | ~66–70 | 70–140/min ≈ 100.000–200.000/Tag |

Dazu kommt: Die hvv-Web-Auskunft rendert Abfahrten per JavaScript. Eine LLM-Websuche liefert in der Regel indexierte bzw. zwischengespeicherte Inhalte und keine Live-Abfahrtstafel ⚠️. Grok kann laut Drittquellen Web und X durchsuchen und zeitgesteuerte „Automations“ ausführen ([hackceleration](https://hackceleration.com/training/grok), [xAI-Provider-Guide](https://preview.hex.pm/preview/req_llm/show/guides/providers/xai.md)). Eine **Smartphone-App bedienen** kann er nicht ⚠️ (offizielle xAI-Doku nicht geprüft). Außerdem kann ein LLM beim Auslesen Zahlen vertauschen oder erfinden. Für eine Pipeline, die Züge zuordnet, ist das ein Ausschlusskriterium. Ein normales Skript ist billiger, schneller und deterministisch. Damit stellt sich aber sofort die Frage nach den Nutzungsbedingungen.

### 1.3 Was sagen die Nutzungsbedingungen?

- **hvv-Entwicklerschnittstelle (Geofox GTI)**, [hvv Datenabruf](https://www.hvv.de/de/fahrplaene/abruf-fahrplaninfos/datenabruf). Wortlaut laut Suchausschnitt: *„Der Nutzer darf die ihm überlassenen Inhalte ausschließlich zum Zwecke der unentgeltlichen Fahrplanauskunft für Endkunden nutzen.“* Außerdem soll die **Serverlast möglichst gering** bleiben, und der hvv begrenzt **Anzahl der Abfragen und Datenvolumen** (sinngemäß aus dem Suchausschnitt, ⚠️ genauer Wortlaut nicht abrufbar). Der Zugang gibt es nur auf Antrag.
- **Abfahrtsmonitor und Verlinkungsservice:** Es gibt eigene Bedingungen. Danach darf die Auskunft z. B. nur „unverändert und eigenständig im eigenen Browser-Fenster“ erscheinen ([Verlinkungsservice](https://www.hvv.de/de/fahrplaene/abruf-fahrplaninfos/fahrplanauskunft-webseite/nutzungsbedingungen-verlinkungsservice), [Abfahrtsmonitor](https://www.hvv.de/de/fahrplaene/abruf-fahrplaninfos/abfahrten-auf-ihrem-monitor/abfahrten-anzeige/nutzungsbedingungen-abfahrtsmonitor)). Die Bedingungen erlauben Einbettung, aber kein Auslesen und Weiterverarbeiten. Der Abfahrtsmonitor wird laut hvv derzeit überarbeitet und ist nicht verfügbar.
- **Allgemeine Nutzungsbedingungen von hvv.de und der hvv-App:** ⚠️ Eine ausdrückliche Klausel zu automatisiertem Abruf, Bots, Scraping oder einen Text-and-Data-Mining-Vorbehalt nach § 44b UrhG habe ich **nicht gefunden**. hvv.de war aus der Sitzung nicht erreichbar, und die Websuche lieferte keinen solchen Text. robots.txt nicht geprüft ⚠️.
- **Gesetzlich unabhängig davon:** Die Abfahrtsdaten sind eine Datenbank. Nach § 87b UrhG ist auch das **wiederholte und systematische** Entnehmen kleiner Teile unzulässig, wenn es der normalen Auswertung der Datenbank zuwiderläuft ([§ 87b UrhG](https://www.gesetze-im-internet.de/urhg/__87b.html)). Ein Dauer-Scraper für eine konkurrierende App ist genau dieser Fall (keine Rechtsberatung). Hinzu kommt der Interessenkonflikt: Der hvv beauftragt die Prüfdienste selbst (siehe sbahn_hamburg_recherche.md, Abschnitt 7).

**Ergebnis:** Ein Scraping von hvv.de/hvv-App in der nötigen Frequenz wäre technisch wacklig, teuer und rechtlich angreifbar. In der seltenen Frequenz wäre es nutzlos. Also nicht machen.

### 1.4 Gibt es einen sinnvollen täglichen Abruf?

Ja, aber **nicht für Verspätungen** und **nicht zwingend mit hvv als Quelle**:

| Täglicher Job | Quelle | Nutzen |
|---|---|---|
| Fahrplanstand prüfen (neue GTFS-Version, Gültigkeit) | hvv-GTFS im Transparenzportal (dl-de/by-2.0) | ist schon in KONZEPT 6.1 vorgesehen |
| Geplante Änderungen für die nächsten Tage erkennen (Sperrungen, SEV, geänderte Linienwege) | DB Timetables `/plan` (enthält Planänderungen), gtfs.de ServiceAlerts, hvv-GTFS `calendar_dates` | Die App zeigt „Fahrplan heute geändert“, sonst stimmen die Kandidaten nicht |
| Redaktionelle Zusammenfassung für den Betreiber („Diese Woche: S3 Harburg–Neugraben gesperrt“) | Pressemeldungen, Baustellenseiten (manuell lesen) | Hier *kann* ein LLM wie Grok helfen: 1 Mal täglich Nachrichten zusammenfassen, **ein Mensch prüft**, dann wird eine Hinweiszeile gesetzt. Kein Scraping von Abfahrtsdaten. |

---

## 2. Datenquellen im Vergleich

| Quelle | Inhalt | Echtzeit S-Bahn HH? | Limit | Lizenz / Bedingungen | Eignung |
|---|---|---|---|---|---|
| **DB Timetables API** (IRIS-basiert), [Marketplace](https://developers.deutschebahn.com/db-api-marketplace/apis/product/timetables) | `/plan/{eva}/{yymmdd}/{hh}` (Soll je Stunde), `/fchg/{eva}` (alle Änderungen), `/rchg/{eva}` (Änderungen der letzten ca. 2 min), `/station/{pattern}`; Zugnummer, Gleis, `pt`/`ct` minutengenau | **ja.** S-Bahn-Hamburg-Abfahrten mit Verspätungen erscheinen in IRIS-Tafeln, z. B. [DBF „Hamburg Hbf (S-Bahn)“](https://dbf.finalrewind.org/Hamburg%20Hbf%20(S-Bahn)) mit S1-Abfahrten und Hinweisen | **60 Aufrufe/min** (Free-Plan) ⚠️ laut [db-timetable-mcp README](https://github.com/jorekai/db-timetable-mcp), die den Marketplace zitiert | **CC BY 4.0** ⚠️ (gleiche Quelle). API-Key und Abo des Produkts nötig | **Erste Wahl für Phase 2.** Nur über eigenen Server-Cache, nie direkt vom Handy |
| **RIS::Stations**, [Marketplace](https://developers.deutschebahn.com/db-api-marketplace/apis/product/ris-stations) | Stationsstammdaten, EVA ↔ Name ↔ Koordinaten | nein (Stammdaten) | laut Ausschnitt **kostenloser Testzugang max. 2 Monate**, 10 Anfragen/s, 10.000 Anfragen ⚠️ (Ausschnitt abgeschnitten) | Daten CC BY 4.0 (lt. [db-hafas-stations license-data](https://www.npmjs.com/package/db-hafas-stations)) | Nicht als Live-API nötig. Die EVA-Liste einmal erzeugen und versionieren (siehe 2.1). Alternative: StaDa ⚠️ nicht geprüft |
| **RIS::Boards / RIS::Journeys** | Abfahrtstafeln bzw. Fahrtverläufe inkl. Vereinigung/Flügelung | ja | Boards: API-Key nötig ([db-vendo-client db-apis.md](https://github.com/public-transport/db-vendo-client/blob/main/docs/db-apis.md)); Konditionen ⚠️ nicht geprüft. Journeys: kostenpflichtig, Vertrag | proprietär | Journeys: unrealistisch. Boards: Konditionen anfragen, falls Timetables nicht reicht |
| **bahnhof.de- / regio-guide.de-Endpunkte** | Tafeln ohne Key | ja | inoffiziell | keine Lizenz | **nicht für Produktion** (inoffiziell, kann jederzeit brechen) |
| **gtfs.de GTFS-RT**, [gtfs.de/realtime](https://gtfs.de/de/realtime/) | TripUpdates + ServiceAlerts, deutschlandweit, alle 10 s, `realtime.gtfs.de/realtime-free.pb` | **⚠️ unklar.** Hamburger Hochbahn: „in Vorbereitung / Klärung von Lizenzfragen“. Eine Quelle „SIRI Bahn (Regionalverkehr)“ ist gelistet. Ob die S-Bahn Hamburg darüber kommt, muss man testen | Feed-Download, kein Abfrage-Limit genannt | CC BY-SA 4.0 | Als Zweitquelle testen. Der Feed passt zur gtfs.de-Statik, nicht zum hvv-GTFS, also Mapping nötig |
| **DELFI-Echtzeit (Mobilithek)**, [Blog](https://mobilithek.info/blog/deutschlandweite-prognosedaten), [Angebot GTFS-RT](https://mobilithek.info/offers/755009281410899968) | Deutschlandweite Prognosedaten als **GTFS-RT TripUpdates und SIRI-ET**, seit 08/2024, seit Ende 2024 offen. Regionaldatensatz „DELFI-Realtime SIRI ET Bremen, Niedersachsen, **Hamburg** und Rostock/VVW“ ([GovData](https://www.govdata.de/suche?state=04&sourceportal=8196e75f-c45e-4281-ac8f-1803c4e1f3be)), technisch über den RegioCluster Nord/VBN ([mobiflip](https://www.mobiflip.de/echtzeitdaten-fuer-ganz-deutschland-das-steckt-hinter-dem-google-maps-deal/amp/)) | **⚠️ möglich, nicht verifiziert.** „Hamburg“ steht im Titel des SIRI-ET-Datensatzes. Ob die DB-S-Bahn vollständig und in Echtzeit drin ist, ist offen | Feed-Download | lt. [Transitous-Feedliste](https://github.com/public-transport/transitous/blob/main/feeds/de.json) CC BY-SA 4.0 (GTFS-RT) bzw. CC BY 4.0 (SIRI-Spiegel) ⚠️ | **Wichtigster Test.** Wäre die S-Bahn drin, ersetzt ein Feed-Download (keine Rate-Limit-Rechnung) die Stations-Abfragen |
| **hvv / Hochbahn offen** | Transparenzportal: nur **Soll**-GTFS (monatlich, dl-de/by-2.0, aktuell bis Dez. 2026) ([Transparenzportal](https://suche.transparenz.hamburg.de/dataset/hvv-fahrplandaten-gtfs-juni-2025-bis-dezember-2025), [Mobility Database](https://mobilitydatabase.org/feeds/gtfs/mdb-3362)). Echtzeit im Portal nur für StadtRAD u. Ä. | **nein** (kein offener hvv-Echtzeitfeed gefunden) | – | – | Nur Sollfahrplan. Geofox GTI: nur auf Antrag, zweckgebunden, für diese App unwahrscheinlich |
| **v6.db.transport.rest** | REST-Wrapper um DB-Navigator-APIs | – | früher 100 Anfragen/min | – | **Abgeschaltet:** „This API is not operational anymore“ ([Doku](https://github.com/derhuerst/db-rest/blob/6/docs/readme.md)). Selbst hosten (db-vendo-client) nutzt inoffizielle APIs mit „much lower rate limit“, also nicht für Produktion |
| **hamburg-gtfs-rt-server** (derhuerst) | Inoffizieller Hamburg-GTFS-RT-Feed | – | – | – | **Deprecated**, weil die HVV-HAFAS-API abgeschaltet wurde ([README](https://github.com/derhuerst/hamburg-gtfs-rt-server)) |
| **Transitous / MOTIS**, [API-Policy](https://transitous.org/api/) | Routing/Abfahrten mit Echtzeit aus offenen Feeds (u. a. DELFI) | nur so gut wie DELFI/gtfs.de (⚠️ siehe oben) | best effort, „should not be flooded“ | **„not intended for commercial or for-profit purposes“**. Gedacht für FOSS und non-profit, kann jederzeit eingestellt werden. Attribution Pflicht | Prototyp: ok. Produktion: nein. Stattdessen **MOTIS selbst hosten** mit denselben Feeds |

### 2.1 EVA-Nummern der Hamburger S-Bahn-Stationen

Quelle: `db-hafas-stations` 2.0.0 (npm), Stationsdaten DB InfraGO, CC BY 4.0. Gefiltert auf „suburban“ im Raum Hamburg, dazu die großen Knoten. Alle Stationen aus sbahn_hamburg_recherche.md Abschnitt 3 sind enthalten (66 Stationen).

| EVA | Station | EVA | Station | EVA | Station |
|---|---|---|---|---|---|
| 8006236 | Wedel(Holst) | 8006705 | Reeperbahn | 8000805 | Barmbek |
| 8005106 | Rissen | 8003518 | Landungsbrücken | 8000512 | Alte Wöhr |
| 8005785 | Sülldorf | 8005663 | Stadthausbrücke | 8005207 | Rübenkamp |
| 8003088 | Iserbrook | 8003137 | Jungfernstieg | 8004661 | Ohlsdorf |
| 8001007 | Blankenese | 8002549 | **Hamburg Hbf** ⚠️ | 8002547 | Hamburg Airport |
| 8002873 | Hochkamp | 8000908 | Berliner Tor | 8003410 | Kornweg (Klein Borstel) |
| 8003312 | Klein Flottbek | 8003519 | Landwehr | 8002946 | Hoheneichen |
| 8004719 | Othmarschen | 8002628 | Hasselbrook | 8006319 | Wellingsbüttel |
| 8000780 | Bahrenfeld | 8006198 | Wandsbeker Chaussee | 8004862 | Poppenbüttel |
| 8002562 | Ottensen | 8002106 | Friedrichsberg | 8002553 | **Altona** ⚠️ |
| 8006706 | Königstraße | 8002548 | Dammtor | 8002550 | Holstenstraße |
| 8070146 | Sternschanze | 8001438 | Diebsteich | 8003541 | Langenfelde |
| 8002559 | Stellingen | 8002555 | Eidelstedt | 8001739 | Elbgaustraße |
| 8003460 | Krupunder | 8002545 | Halstenbek | 8005858 | Thesdorf |
| 8004819 | Pinneberg | 8004266 | Hammerbrook | 8002551 | Elbbrücken |
| 8006062 | Veddel | 8002561 | Wilhelmsburg | 8000147 | **Harburg** ⚠️ |
| 8004267 | Harburg Rathaus | 8006749 | Heimfeld | 8006750 | Neuwiedenthal |
| 8002557 | Neugraben | 8002556 | Fischbek | 8004247 | Neu Wulmstorf |
| 8001315 | Buxtehude | 8004302 | Neukloster (Kr Stade) | 8003002 | Horneburg |
| 8001493 | Dollern | 8000434 | Agathenburg | 8000089 | Stade |
| 8005191 | Rothenburgsort | 8005870 | Tiefstack | 8000971 | Billwerder-Moorfleet |
| 8004048 | Mittlerer Landweg | 8000779 | Allermöhe | 8004241 | Nettelnburg |
| 8002554 | Bergedorf | 8005016 | Reinbek | 8006541 | Wohltorf |
| 8000669 | Aumühle | | | | |

⚠️ **Hbf, Altona, Harburg:** In IRIS gibt es eine eigene Tafel **„Hamburg Hbf (S-Bahn)“** ([DBF](https://dbf.finalrewind.org/Hamburg%20Hbf%20(S-Bahn))). Wahrscheinlich laufen die S-Bahn-Gleise dort unter einer **eigenen EVA-Nummer**, die von 8002549 abweicht. Dasselbe ist für Altona (S) und Harburg (S) zu erwarten. Vor dem Bau mit `/station/Hamburg Hbf` usw. prüfen, welche EVA die S-Bahn-Abfahrten liefert.

Hinweis zur Zählung: Nach der Stationsliste in sbahn_hamburg_recherche.md hat die S1 **31** Stationen (26 Stamm + Airport + 4 Poppenbüttel-Ast). Die Angabe „36 Stationen“ in der Aufgabe bzw. im Recherchedokument („36 bzw. 30 Stationen“) passt nicht dazu. Ich rechne unten mit beiden Werten.

---

## 3. Rechnung: Abfrageintervalle mit der DB Timetables API

### Annahmen
- Limit 60 Aufrufe/min ⚠️ (nur über Zweitquelle belegt).
- `/plan` braucht man pro Station und Stunde einmal, plus die Folgestunde vorab. Das sind ~1–2 Aufrufe pro Station und Stunde. Bei 70 Stationen sind das ≤ 140/h ≈ **2,3/min**. Die Planwerte lassen sich auch komplett nachts für den ganzen Tag holen (70 × 24 = 1.680 Aufrufe in ~30 min).
- `/fchg` einmal beim Start bzw. nach Fehlern, danach `/rchg`. `/rchg` liefert nur die Änderungen der **letzten ca. 2 min**. Jede Station muss deshalb **öfter als alle 2 min** abgefragt werden, sonst gehen Änderungen verloren (dann wieder `/fchg`).
- Für `rchg` bleiben etwa **57 Aufrufe/min**, mit Sicherheitsreserve ~50/min (so auch KONZEPT 6.4).

### Variante A: jede Station reihum

| Umfang | Stationen | Intervall je Station (57/min) | Intervall je Station (50/min, mit Reserve) | `rchg`-Fenster eingehalten? |
|---|---|---|---|---|
| S1 (laut Liste) | 31 | 33 s | 37 s | ja |
| S1 (Aufgabenwert) | 36 | 38 s | 43 s | ja |
| ganzes Netz | 66 | 69 s | 79 s | ja, knapp (Reserve ~40 s) |
| ganzes Netz (Aufgabenwert) | 70 | 74 s | 84 s | ja, knapp |

### Variante B: Stützstationen (empfohlen)

Eine Station liefert in `fchg`/`rchg` die **Prognose für alle Züge, die sie noch anfahren**. Die aktuelle Verspätung eines Zuges sieht man also schon an der nächsten abgefragten Station vor ihm. Man muss nicht jede Station abfragen, sondern nur so viele, dass jeder Zug immer eine abgefragte Station in den nächsten ~5–10 min vor sich hat:

- **Stützstationen (Vorschlag, 14):** Wedel, Blankenese, Altona (S), Hbf (S), Berliner Tor, Barmbek, Ohlsdorf, Poppenbüttel, Airport, Eidelstedt, Harburg (S), Neugraben, Bergedorf, Buxtehude.
- 14 Stationen × alle **15 s** = 56/min (zu knapp) → **alle 20 s** = 42/min. Es bleibt Reserve für `/plan` und `/fchg`-Resyncs.
- Nur S1: 7–8 Stützstationen alle 10–15 s ≈ 35–45/min.

### Reicht das für die Zuordnung?

- **Fehlerbudget:** Die DB-Zeiten sind **minutengenau** (`ct` = `YYMMddHHmm`), das ergibt ±30 s Rundung. Dazu kommen die Latenz der DB-Prognose (⚠️ unbekannt, erfahrungsgemäß eher 1–2 min nach dem Ereignis) und das Polling-Alter (im Mittel halbes Intervall, 10–40 s). Realistisch bleibt ein Positionsfehler von **±1–1,5 min**.
- **S1 im 5-Minuten-Takt** (in der HVZ, sonst 10 min mit Flügelung): Dieser Fehler liegt klar unter dem halben Abstand (2,5 min). **Das reicht.**
- **City-Tunnel im 2,5-Minuten-Takt:** Der halbe Abstand ist 1,25 min. Das liegt im Fehlerbereich, die DB-Daten allein **reichen nicht sicher**. Lösbar ist es trotzdem:
  - Reihenfolge-Regel: kein Überholen (KONZEPT 6.2).
  - Im Tunnel fahren S1 und S3 (bzw. S7) abwechselnd mit **unterschiedlichen Zielen**. Die Frage „Was steht am Zug?“ (KONZEPT 6.3) trennt sie sofort.
  - Crowd-Durchfahrten (6.4) korrigieren die Position auf Sekunden.
- **Ausfälle** (`rchg`/`fchg` mit Status „c“) sind für die Zuordnung mindestens so wichtig wie Verspätungen: Fällt ein Zug aus, verschiebt sich sonst jede Zuordnung um einen Takt.

---

## 4. Kostencheck Supabase Realtime

### Aktuelle Preise (Supabase Pro)
Quelle: [Realtime Pricing](https://supabase.com/docs/guides/realtime/pricing), [Manage Realtime Messages usage](https://supabase.com/docs/guides/platform/manage-your-usage/realtime-messages) (über Suchausschnitte, ⚠️ Seite nicht direkt geöffnet)

| Posten | Free | Pro |
|---|---|---|
| Grundpreis | 0 $ | 25 $/Monat (mit Compute-Guthaben für die kleinste Instanz) |
| Realtime-Nachrichten inkl. | 2 Mio. (keine Überschreitung abrechenbar, sondern Drosselung) | 5 Mio., danach **2,50 $ je 1 Mio.**, abgerechnet in ganzen Paketen |
| Gleichzeitige Verbindungen (Peak) inkl. | 200 ⚠️ (nicht bestätigt) | 500, danach **10 $ je 1.000** |

**Zählweise:** Jeder Empfänger zählt. Ein Broadcast an 4 Clients ergibt 5 Nachrichten (1 gesendet + 4 empfangen). Eine DB-Änderung mit 5 Zuhörern ergibt 5 Nachrichten.

### Rechnung des Reviewers: Push alle 30 s an jeden Nutzer

- 1.000 Nutzer × (8 h × 120 Nachrichten/h = 960/Tag) = 960.000/Tag × 30 Tage = **28,8 Mio./Monat**. **Arithmetisch korrekt.** Die gesendeten Nachrichten (1 je Broadcast und Kanal, z. B. 5 Linienkanäle × 960/Tag × 30 ≈ 0,14 Mio.) fallen kaum ins Gewicht.
- Nachrichten: 28,9 Mio. − 5 Mio. = 23,9 Mio. → 24 Pakete × 2,50 $ = **60 $**
- Verbindungen: 1.000 Peak − 500 = 500 → 1 Paket = **10 $** ⚠️ (Paketlogik bei Verbindungen nicht bestätigt)
- **Summe: ca. 95 $/Monat** (25 + 60 + 10). Das ist nicht ruinös, aber 4× der Plan im KONZEPT (25 $). Es wächst linear mit Nutzern und Push-Frequenz.

### Variante: nur Änderungen pushen

Annahme: 300 relevante Ereignisse/Tag im Netz (Meldungen, Entwarnungen, Korrekturen, Ausfälle). Jedes Ereignis geht an die Abonnenten seiner Linie. Bei 1.000 Nutzern, die im Schnitt 1,5 von 5 Linien abonnieren, erreicht ein Ereignis ~300 Clients.

- 300 × 300 × 30 = **2,7 Mio. Nachrichten/Monat**, also unter den 5 Mio. inklusive → 0 $.
- Selbst bei 1.000 Ereignissen/Tag (9 Mio.) wären es nur 2 Pakete = 5 $.
- Verbindungen bleiben bei 1.000 → 10 $.
- **Summe: ca. 35 $/Monat** (−63 %).
- **Wichtig:** Verspätungsupdates nicht als einzelne Realtime-Nachrichten verschicken (bei ~60–80 fahrenden Zügen und Updates je Minute wäre man schnell wieder bei Dutzenden Millionen). Die gehören in den Snapshot.

### Variante: CDN-Snapshot alle 15–30 s pollen

Der Server (Echtzeit-Worker) schreibt alle 15 s eine kleine `live.json` (Verspätungen, Ausfälle, aktuelle bereinigte Meldungen) auf ein CDN. Die App holt sie mit `If-None-Match`/ETag.

| Intervall | Requests/Monat (1.000 Nutzer × 8 h) | Datenmenge bei 10 KB gzip (ohne 304-Ersparnis) |
|---|---|---|
| 30 s | 28,8 Mio. | ~290 GB |
| 15 s | 57,6 Mio. | ~580 GB |

- Supabase Realtime: **0 Nachrichten, 0 Verbindungen** → nur 25 $ Pro-Grundpreis (fürs Melden/Edge Functions).
- CDN: Bei einem CDN mit kostenlosem bzw. sehr günstigem Traffic ⚠️ (Preise nicht geprüft; grob 0–6 $ bei ~0,01 $/GB) kommt man auf **ca. 25–35 $/Monat** gesamt. **GitHub Pages ist dafür ungeeignet:** Es hat ein weiches Bandbreitenlimit von ~100 GB/Monat ⚠️ und ist nicht für Live-Daten gedacht.
- Mit ETag/304 und Deltas sinkt der Traffic stark, weil sich der Snapshot oft nicht ändert.
- Nachteil: im Mittel +7,5–15 s Verzögerung. Für Kontrollmeldungen ist das vertretbar, das KONZEPT verzögert ohnehin bewusst. Für Verspätungen ist es egal.
- Nebeneffekt: Ein CDN-Snapshot ist ausfallsicher (KONZEPT 7, Notbetrieb über `status.json`) und skaliert ohne Verbindungslimit.

**Kostenvergleich (1.000 gleichzeitige Nutzer, 8 h/Tag)**

| Modell | Supabase | CDN | Summe/Monat |
|---|---|---|---|
| Push alle 30 s (Reviewer) | 95 $ | – | **~95 $** |
| Push nur Änderungen | ~35 $ | – | **~35 $** |
| Snapshot-Polling 15–30 s | 25 $ | 0–10 $ ⚠️ | **~25–35 $** |
| Hybrid: Meldungen per Push, Verspätungen per Snapshot | ~35 $ | 0–10 $ | **~35–45 $** |

---

## 5. Empfehlung

1. **Grok-Abruf der hvv-App verwerfen.** Allenfalls ein täglicher, von einem Menschen geprüfter Nachrichten-Überblick zu Baustellen. Geplante Änderungen maschinell aus hvv-GTFS, DB `/plan` und ServiceAlerts holen.
2. **Phase 0/1 (jetzt, 1 Tag Aufwand):**
   - Test A: DELFI-GTFS-RT ([Mobilithek](https://mobilithek.info/offers/755009281410899968)) und `realtime-free.pb` von gtfs.de herunterladen. Filtern, ob TripUpdates für S1/S3/S5 Hamburg mit plausiblen Verspätungen vorkommen (z. B. mit [print-gtfs-rt-cli](https://github.com/derhuerst/print-gtfs-rt-cli)). Über einen Tag hinweg mit DBF/IRIS vergleichen.
   - Test B: Key für die DB Timetables API holen. Limit, Lizenztext und die **S-Bahn-EVA für Hbf, Altona, Harburg** per `/station/` prüfen.
3. **Phase 2:**
   - Wenn DELFI die S-Bahn HH zuverlässig enthält: Feed alle 30 s ziehen (ein Download statt Stations-Abfragen) und auf `fahrtKey` mappen. DB Timetables als Fallback und für Ausfälle und Gleise.
   - Sonst: DB-Timetables-Worker mit **14 Stützstationen alle 20 s** (`rchg`), nachts `/plan` für den Tag, `/fchg` bei Resync.
   - In beiden Fällen gilt: Im City-Tunnel entscheiden Zielabfrage, Reihenfolge-Regel und Crowd-Durchfahrten, nicht die DB-Minute allein.
4. **Verteilung:** **Hybrid.** Kontrollmeldungen als bereinigte Ereignisse per Supabase Realtime (nur Änderungen, Linienkanäle). Verspätungen und Ausfälle als CDN-Snapshot (`live.json`, 15–30 s, ETag). Ausgabenlimit in Supabase aktiv lassen. KONZEPT 7 („ca. 25 $“) ist damit realistisch, bei 1.000 gleichzeitigen Nutzern eher 35–45 $.
5. **Nicht einplanen:** Geofox GTI (Zweckbindung, Freigabe unwahrscheinlich), v6.db.transport.rest (abgeschaltet), db-vendo-client oder bahnhof.de-Endpunkte (inoffiziell), Transitous als Produktions-Backend (nicht für kommerzielle Nutzung). MOTIS selbst zu hosten ist eine Option, falls man Routing braucht.

### Offene Punkte ⚠️
- Wortlaut der allgemeinen Nutzungsbedingungen von hvv.de/hvv-App zu automatisiertem Abruf (Seite nicht erreichbar, kein Text gefunden).
- Rate-Limit und Lizenz der DB Timetables API direkt auf dem Marketplace bestätigen. Außerdem prüfen, ob die Marketplace-Bedingungen eine solche Nutzung einschränken.
- Abdeckung der S-Bahn Hamburg in DELFI-Echtzeit und gtfs.de.
- Eigene S-Bahn-EVA-Nummern an Hbf, Altona, Harburg.
- Latenz der DB-Prognosen für die S-Bahn Hamburg (messen: DB-`ct` vs. Crowd-Durchfahrt).
- Supabase-Verbindungspakete und Free-Limit von 200 Verbindungen auf der Live-Preisseite bestätigen.
