# Lösungen K: Verhalten der Kontrolleure

Bezug: [`04_kontrolleur_verhalten.md`](04_kontrolleur_verhalten.md), [`../KONZEPT.md`](../KONZEPT.md), [`../recherche/sbahn_hamburg_recherche.md`](../recherche/sbahn_hamburg_recherche.md).
Die Bausteine **Q1–Q6** werden einmal am Ende unter „Querschnitt“ definiert, die Szenarien verweisen darauf.

**Faktenkorrekturen an den Szenarien** (nach der Recherche):
- **K-03:** Altona–Holstenstraße fährt derzeit nur die **S7, und nur Richtung Aumühle** (Brücke einseitig). Die S5 fährt nicht über Altona, die S3 nicht über Holstenstraße. Realistisches Pendeln ist hier ein **Dreieck**: Altona → Holstenstraße (S7), Holstenstraße → Diebsteich (S5/S2), Diebsteich → Altona (S3). Die Lösung unten deckt genau das ab.
- **K-05:** Um 13:12 fährt **keine S2** (nur HVZ 6–9 und 15–19 Uhr). Richtung Bergedorf fährt dann die **S7**.
- **K-06:** Jungfernstieg bedienen **S1, S3, S7**. Die S2 hält dort nicht (sie fährt Hbf–Dammtor). Die S7 fehlt in der Aufzählung.
- **K-09:** Eine **S21 gibt es seit Dezember 2023 nicht mehr** (heute S2). Eidelstedt bedienen S3 und S5. Variante 2 sinngemäß: Falschmeldungen auf der S5, echtes Team auf der S3.
- **K-01:** Blankenese liegt +13 min hinter Wedel, also 08:16 statt 08:15. Für die Lösung egal.

---

### K-01 Zivilkontrolleure, erst beim „Fahrausweise bitte“ erkennbar
- **Lösung:**
  1. **Sofortwarnung im selben Zug.** Ein `IM_ZUG` geht per Realtime an alle Geräte, die in derselben Fahrt (bzw. demselben physischen Zug) eingecheckt sind, mit Vibration und Banner („Kontrolle in deinem Zug, bemerkt in Blankenese, Einheit vorne“). Das ist der einzige Moment, in dem die Meldung noch Mitfahrenden in anderen Wagen nützt.
  2. **Wortlaut ändern:** Auf der Zugkarte heißt der rote Rand **„bemerkt in Blankenese“**, nicht „eingestiegen in“ (KONZEPT 4.2). Die App behauptet nicht, wann sie eingestiegen sind.
  3. **Rückwirkende Angabe** als optionaler Nachtrag-Chip (Q3): „Waren schon vorher da“ → Auswahl der vorherigen Halte dieser Fahrt oder „seit Abfahrt“. Feld `seit_station_id`. Die Zugkarte zeichnet ab dort eine **gestrichelte rote Linie** („vermutlich seit Wedel“). Das Team-Objekt (Q1) bekommt damit den richtigen Startpunkt, was die Vorhersage und die aggregierte Statistik verbessert.
  4. **Lerneffekt bekämpfen:** Status „unbeobachtet“ und Beobachtungsdichte (Q4). Ein Zug ohne eingecheckte Nutzer zeigt nicht ⚪, sondern ▒ „unbeobachtet – niemand meldet aus diesem Zug“. Einmaliger Hinweis beim ersten Start: „Kontrolleure in Zivil werden erst bei der Kontrolle erkennbar. Keine Meldung heißt nicht keine Kontrolle.“
- **Status:** 🟡 teilweise. Der Kern („warnen, bevor sie sich zu erkennen geben“) ist ❌ nicht lösbar: Ohne Erkennungsmerkmal gibt es keine Meldung, und Personenbeschreibungen sind bewusst ausgeschlossen.
- **Restrisiko:** Wer vor der ersten Kontrolle eingestiegen ist, bleibt ungewarnt. Die App macht das sichtbar, indem sie nie Grün zeigt, „unbeobachtet“ von „beobachtet, nichts gemeldet“ trennt und rückwirkende Linien gestrichelt (= geschätzt) zeichnet.

### K-02 Team teilt sich auf zwei Zugteile auf (Flügelzug S1)
- **Lösung:**
  1. **Zugteil automatisch statt optional.** Beim Check-in in eine geflügelte S1 vor Ohlsdorf fragt die App einmal: „Fährst du zum **Airport (vorne)** oder nach **Poppenbüttel (hinten)**?“. Das ist das Fahrtziel, das der Nutzer ohnehin kennt. Daraus folgt `einheit` (1 = vorne, 2 = hinten), und alle Meldungen dieses Geräts tragen sie automatisch. Ohne Antwort bleibt `einheit = null`.
  2. **Einheitliches Feld `einheit`** statt `zugteil` (Q3). Bei der S1 gilt Einheit 1 = Airport, Einheit 2 = Poppenbüttel (KONZEPT 5.4 bleibt sonst gültig).
  3. **Der andere Teil wird nicht ⚪.** Hat nur Einheit 1 eine Meldung, bekommt Einheit 2 vor und nach Ohlsdorf den Status **◌ „möglich“** („Teams teilen sich manchmal auf“), solange dort niemand aus Einheit 2 aktiv „bei uns nichts“ gemeldet hat bzw. als Beobachter eingecheckt ist (Q4). Ist Einheit 2 beobachtet und ohne Meldung, zeigt sie ⚪ mit „beobachtet“.
  4. **Team-Objekt (Q1):** Meldungen aus beiden Einheiten desselben physischen Zuges gehören zu **einem** Team mit Einheiten-Menge `{1, 2}`. Nach Ohlsdorf wird das Team geteilt: jede Teilfahrt bekommt den Teil mit ihren Einheiten. Bei der Vereinigung in der Gegenrichtung werden beide Teams wieder zusammengeführt.
- **Status:** 🟡 teilweise
- **Restrisiko:** Sitzt im hinteren Teil niemand mit App, weiß die App nichts über F3/F4. Sie zeigt dann ehrlich ◌ „möglich, Teil unbeobachtet“ statt ⚪. Die Annahme, dass Teams den Zugteil selten wechseln, ist laut Recherche plausibel, aber unbelegt; die Gewichte müssen mit echten Daten nachjustiert werden.

### K-03 Pendeln zwischen zwei Stationen
- **Lösung:**
  1. **Bereich vom Typ `PENDEL`** (Q1). Wird ein Team innerhalb von 45 min mindestens 3-mal in mindestens 2 verschiedenen Fahrten (oder Fahrt + Bahnsteig) gemeldet, und liegen alle Orte in einem zusammenhängenden Teilgraphen von höchstens 4 Stationen, entsteht ein Bereich. Seine Stationen sind alle Knoten auf kürzesten Wegen zwischen den gemeldeten Orten im **linienübergreifenden Netzgraphen** (Kanten = direkte Zugverbindungen aus GTFS, gerichtet). Das Dreieck Altona → Holstenstraße → Diebsteich → Altona ist damit ein Bereich aus 3 Stationen und 3 gerichteten Kanten über S7, S5/S2 und S3.
  2. **Dauer:** Der Bereich gilt bis 30 min nach der letzten zugehörigen Meldung, jede neue Meldung verlängert ihn. Danach 15 min 🟠 „vermutlich vorbei“.
  3. **Darstellung:** In der Linienübersicht (4.3) und auf jeder Zugkarte (4.2) liegt ein **roter Balken** über den betroffenen Stationen und Kanten, auf allen Linien, die dort fahren: „Abschnitt besetzt seit 10:05 · 7 Meldungen“. Auf der Abfahrtstafel (4.1) bekommt jeder Zug, dessen restlicher Weg durch den Bereich führt, das Etikett „fährt durch besetzten Abschnitt“, **auch wenn der Zug selbst ⚪ ist**.
  4. Einzelmeldungen im Bereich werden nicht mehr einzeln hervorgehoben, sondern als Zähler am Bereich angezeigt.
- **Status:** ✅ gelöst
- **Restrisiko:** Bis zur dritten Meldung (ca. 15–30 min) ist das Muster unsichtbar. Pendelt das Team über mehr als 4 Stationen, entsteht kein Bereich, nur ein wanderndes Team mit Vorhersage. Die App zeigt beim Bereich „seit“ und „zuletzt bestätigt“, damit Nutzer das Alter einschätzen können.

### K-04 Abgangskontrolle direkt nach dem Ausstieg
- **Lösung:**
  1. **Sofortbanner an Mitfahrende:** `AUSGESTIEGEN_IN Harburg Rathaus` geht sofort an alle Eingecheckten derselben Fahrt **und** an Eingecheckte in Zügen, die in den nächsten 15 min in Harburg Rathaus halten: „Team gerade in Harburg Rathaus ausgestiegen – Abgangskontrolle möglich“.
  2. **Vorwarnung, solange das Team im Zug ist:** Auf der Fahrtleiste (4.5) steht bei 🔴-Zügen dauerhaft „Beim Aussteigen auf Abgangskontrolle achten“. Die Vorhersage (Q1) nennt „bleibt am Bahnsteig“ als häufigste Folge eines Ausstiegs.
  3. **Ausgang statt ganze Station:** Optionaler Nachtrag-Chip „an welchem Ausgang?“ (Feld `ausgang_id`). Die Ausgänge kommen aus GTFS `stops.txt` mit `location_type = 2`, falls der hvv-Datensatz sie enthält (⚠️ nicht geprüft), sonst aus einer kleinen, selbst gepflegten Liste für die ca. 15 größten Stationen (eigene Namen, keine offiziellen Pläne). Die Station bleibt markiert, aber als **halb gefüllter Punkt** mit „Ausgang X“; auf der Abfahrtstafel steht der Ausgang im Text.
  4. `AUSGESTIEGEN_IN` setzt den Zug nicht sofort auf ⚪, sondern 2 Halte lang auf 🟠 „Nachlauf“ (Q2), weil auch ein Teil des Teams im Zug geblieben sein kann.
- **Status:** 🟡 teilweise
- **Restrisiko:** Wer im selben Moment aussteigt wie das Team, ist mit keiner App mehr zu warnen; die Meldung kommt Sekunden zu spät. Ohne Ausgangsangabe bleibt die ganze Station markiert. Die App schreibt dann „Ausgang unbekannt“, statt so zu tun, als wären alle Ausgänge betroffen.

### K-05 Wechsel auf eine andere Linie
- **Lösung:**
  1. **Team wandert über Linien (Q1).** Kommt eine neue Meldung in einer anderen Fahrt, die vom letzten Team-Zustand aus erreichbar ist (Umstieg an einem Knoten, an dem die alte Fahrt ankam und die neue danach abfuhr, mit 2 min Umstiegszeit), wird sie dem Team zugeordnet, sofern das eindeutig ist. Dann wird die alte Fahrt sofort auf 🟠 „Team vermutlich umgestiegen in Berliner Tor“ gesetzt und endet nach 1 Halt. Das löst die falsche Warnung in der S1.
  2. **Vorhersage an Umstiegsknoten (Q1):** Ist die S1-Meldung 2 Halte alt und unbestätigt, und erreicht der Zug einen Knoten (Berliner Tor: S1, S7, S2 in der HVZ, U-Bahn), bekommen die nächsten Abfahrten anderer Linien an diesem Knoten den Status ◌ „möglich“ („Team könnte hier umgestiegen sein“), höchstens 3 Fahrten, 20 min lang.
  3. **Wechsel in die U-Bahn** (Ohlsdorf → U1): Umsteigeknoten zu U-Bahn kennt das Datenskript aus demselben hvv-GTFS (gleiche `parent_station` oder ≤ 300 m). Die App zeigt am Team: „Team hat die S-Bahn möglicherweise verlassen (U1 in Ohlsdorf)“ und beendet die S1-Meldung schneller.
- **Status:** 🟡 teilweise
- **Restrisiko:** Ohne neue Meldung bleibt der Linienwechsel eine Vermutung (◌). Bei mehreren Teams in der Nähe kann die Zuordnung falsch sein; die App ordnet dann nicht zu und zeigt zwei Teams. Die U-Bahn bleibt bis Phase 3 (U-Bahn-Ausbau) ein blinder Fleck, der als solcher benannt wird.

### K-06 Großkontrolle Hbf / Jungfernstieg
- **Lösung:**
  1. **Neuer Melde-Typ:** Im Meldedialog (4.4) ein dritter Punkt „Großkontrolle (viele Kontrolleure)“ → `AN_STATION` mit `gross = true`.
  2. **Bereich vom Typ `GROSSLAGE`** (Q1), automatisch, wenn innerhalb von 15 min (a) mindestens 2 Geräte `gross = true` an Stationen im Umkreis von 2 Kanten melden, oder (b) Meldungen von mindestens 6 verschiedenen Geräten aus mindestens 2 Teams im Umkreis von 2 Kanten um eine Station eingehen. Stationen = alle gemeldeten Stationen plus kürzeste Wege dazwischen (hier Hbf, Jungfernstieg).
  3. **Lange Dauer:** mindestens 90 min ab Beginn; jede Meldung im Bereich verlängert bis „jetzt + 30 min“. Ende erst, wenn 90 min erreicht und 30 min ohne Meldung vergangen sind, danach 30 min 🟠 „vermutlich beendet“. Stationsmeldungen im Bereich verfallen nicht nach 45 min, solange die Großlage läuft.
  4. **Darstellung:** Banner auf allen Screens, deren Station oder Linie den Bereich berührt: „Großkontrolle Hbf / Jungfernstieg seit 16:00 · zuletzt bestätigt 16:42“. Alle Fahrten, deren restlicher Weg durch den Bereich führt (S1, S2, S3, S5, S7), bekommen das Etikett „fährt durch Großkontrolle“. Die Züge im Bereich werden nicht einzeln rot gefärbt, um die Karte lesbar zu halten.
  5. **Filter:** Meldungen innerhalb einer laufenden Großlage gelten als Bestätigungen der Lage und werden nie als Duplikat verworfen (Q5). Serverlast: 60 Zeilen in 10 min sind für Supabase trivial; das Rate-Limit pro Gerät (1 / 10 min) bleibt.
- **Status:** ✅ gelöst
- **Restrisiko:** Die ersten 5–10 min sieht man nur Einzelmeldungen. Die Grenzen des Bereichs sind geschätzt (gemeldete Stationen plus Verbindungen). Die App nennt die Stationen ausdrücklich und schreibt „Bereich geschätzt aus N Meldungen“.

### K-07 Nur eine Station mitgefahren
- **Lösung:**
  1. **Halte-basierter Verfall (Q2)** statt 15/30 min: 🔴 bis 2 Halte nach der letzten Meldung oder Bestätigung (mindestens 4 min), danach 🟠 bis 3 weitere Halte, dann ⚪. Hier: 🔴 bis Hbf, 🟠 bis ca. Elbbrücken. Statt 45 Minuten Fehlwarnung bleiben ca. 10.
  2. **Haltepunkt-Abfrage:** Am ersten Halt nach der Meldung (Dammtor) fragt die Fahrtleiste den Melder mit zwei großen Buttons: „Kontrolleure noch im Zug?“ [Ja, noch da] [Hier ausgestiegen]. Ein Tap erzeugt `BESTÄTIGT` bzw. `AUSGESTIEGEN_IN Dammtor`. Keine Antwort = keine Wirkung.
  3. **„Möglich ausgestiegen“:** Jede Station, die ein 🔴-Zug ohne neue Bestätigung passiert, bekommt 15 min lang ◌ „Team ist hier möglicherweise ausgestiegen“. Die Vorhersage (Q1) wertet das als Kandidat „am Bahnsteig“. Dammtor ist damit markiert, wenn auch nur als „möglich“.
- **Status:** ✅ gelöst
- **Restrisiko:** Antwortet der Melder nicht, ist Dammtor nur ◌ markiert und der Zug noch 2 Halte zu Unrecht rot. Die Zugkarte zeigt „zuletzt bestätigt 07:41 bei Sternschanze“, damit das Alter der Meldung sichtbar ist.

### K-08 Langzug S3 mit drei Einheiten ohne Durchgang
- **Lösung:**
  1. **Zugbildung als Daten:** Das Datenskript (KONZEPT 6.1) schreibt pro Fahrt `einheiten_erwartet`: S3 Mo–Fr ca. 5–19 Uhr = 3, geflügelte S1 bis Ohlsdorf = 2, sonst `null` (unbekannt, 1–2). Die Umläufe sind nicht öffentlich ⚠️, das ist also ein Standardwert.
  2. **Meldung mit Einheit:** Nachtrag-Chip (Q3) „wo im Zug?“ [vorne] [Mitte] [hinten] (in Fahrtrichtung). Bei 3 Einheiten → 1/2/3, bei 2 → vorne = 1, hinten = 2, Mitte = unbekannt. Wer beim Check-in seinen Teil angibt, meldet automatisch mit Einheit.
  3. **Status pro Einheit:** Die Zugkarte zeigt den Zug als 3 Kästchen `[■][□][□]`. Gemeldete Einheit 🔴. Weil zwischen den Einheiten **kein Durchgang** besteht (474 und 490 haben Übergänge nur innerhalb einer Einheit), kann ein Team nur am Halt die Einheit wechseln: Nach jedem Halt ohne neue Meldung werden die Nachbareinheiten ◌ „möglich“. Eine zweite Meldung in Einheit 3 macht diese 🔴 und gehört zum selben Team (gleicher physischer Zug, Q1).
  4. Ohne Einheitsangabe bleibt der ganze Zug 🔴 wie bisher.
- **Status:** 🟡 teilweise
- **Restrisiko:** F2 in Einheit 3 bleibt unsichtbar, solange dort niemand meldet. Wer auf Grund der Anzeige die Einheit wechselt, kann gerade in die unbeobachtete Einheit geraten. Die App kennzeichnet unbeobachtete Einheiten als ▒ (Q4) und zeigt nie „Einheit 3 frei“. Ist die erwartete Zuglänge falsch (Bauarbeiten, Ausfall), passen „Mitte“-Meldungen nicht; dann fällt die App auf „ganzer Zug“ zurück.

### K-09 Kontrolleure kennen die App und ändern ihr Muster
- **Lösung:**
  1. **Variante 1 (Schein-Ausstieg):** `AUSGESTIEGEN_IN` entwarnt nie sofort (Q2): Der Zug bleibt 2 Halte 🟠 „angeblich ausgestiegen“. ⚪ nur, wenn ein zweites, unabhängiges Gerät den Ausstieg bestätigt **oder** der Melder nach der Abfahrt die Haltepunkt-Abfrage mit „nicht wieder eingestiegen“ beantwortet. Ein späteres `IM_ZUG` in derselben Fahrt hebt den Ausstieg auf und macht sofort 🔴 („wieder eingestiegen“).
  2. **Variante 2 (organisierte Falschmeldungen):** Gewichte nach Gerät (Q5): neue Geräte (< 7 Tage oder < 3 bestätigte Meldungen) zählen 0,3; die Summe aller Geräte mit niedrigem Vertrauen ist pro Objekt auf 1,0 gedeckelt („Neulings-Deckel“). Eine Welle frischer Geräte kann so höchstens 🟠 erzeugen. Falsche Warnungen kosten Nutzer nur Zeit; falsche Entwarnungen kosten Geld. Deshalb wiegen Entwarnungen grundsätzlich halb so viel wie Warnungen (Asymmetrie, Q5).
  3. **Variante 3 (späte Kontrolle):** Nachtrag-Chip „kontrollieren gerade?“ [ja] [nein, nur gesehen] (Feld `aktiv`). „Nur gesehen“ ergibt 🟠 „an Bord, kontrollieren noch nicht“; das verfällt nicht, solange das Team gesehen wird. Damit wird auch eine Meldung vor der ersten Kontrolle nützlich.
  4. **Mitlesen durch den Betreiber:** Nicht verhinderbar und kein Problem, solange die App nichts Verdecktes enthält. Wichtig ist nur, dass sie keine personenbezogenen Daten über Kontrolleure erzeugt (Q6).
- **Status:** 🟡 teilweise
- **Restrisiko:** Ein Gegner mit mehreren eingecheckten Altgeräten im echten Zug ist von echten Meldern nicht zu unterscheiden. Die App zeigt die Herkunft jeder Bewertung („2 Meldungen, 1 Entwarnung, davon 1 von neuem Gerät“), damit Nutzer selbst gewichten können. Die Nachlauf-Phase kostet ehrlichen Nutzern 2 Halte Fehlwarnung.

### K-10 vhh-Prüfteam im Umland, in Uniform, die niemand kennt
- **Lösung:**
  1. **Melden trotz Zweifel:** Option „unklar“ (Q3, `sicher = false`). Wer die Westen nicht kennt, kann schon in Thesdorf melden („Personal im Zug, unklar“) statt erst bei der Kontrolle in Pinneberg.
  2. **Fahrtende überträgt statt zu löschen (Q2):** Endet eine Fahrt mit 🔴/🟠 oder wurde in den letzten 2 Halten gemeldet, dann (a) bekommt der Endbahnhof 20 min 🟠 „Team zuletzt hier“, und (b) die **Wendefahrt** bekommt ◌ „Team vermutlich im Gegenzug“ bis 3 Halte nach Abfahrt. Die Wendefahrt bestimmt das Datenskript: gleiche Linie, Gegenrichtung, erste Abfahrt am Endbahnhof ≥ Ankunft + 3 min (GTFS hat keine `block_id`, ⚠️ Heuristik). Hier: nächste S3 ab Pinneberg Richtung Neugraben.
  3. **Rückwirkend:** Nachtrag „waren schon seit Thesdorf da“ (`seit_station_id`) wie in K-01.
  4. **Abdeckung zeigen (Q4):** Auf Außenästen (Pinneberg–Elbgaustraße, Neugraben–Stade, Bergedorf–Aumühle) ohne eingecheckte Beobachter steht ▒ „unbeobachtet – hier melden kaum Nutzer“ statt ⚪. Dazu eine neutrale, feste Info am Ast: „Seit 2026 kontrollieren hier auch hvv-Prüfteams“ (Fakt aus der Recherche, keine Beschreibung von Kleidung oder Personen).
- **Status:** 🟡 teilweise
- **Restrisiko:** Bei wenigen Nutzern im Umland bleibt die App dort meist blind. Sie sagt das aber deutlich (▒), statt ⚪ anzuzeigen. Die Wendefahrt-Heuristik kann falsch sein, wenn der Zug abgestellt oder das Team ausgestiegen ist; deshalb nur ◌, nie 🔴.

### K-11 Kontrolleure nach Feierabend unterwegs
- **Lösung:**
  1. **„Kontrollieren gerade?“ (Q3, `aktiv`):** Melden zwei Geräte „nur gesehen“ und niemand „kontrollieren“, zeigt der Zug 🟠 „Personal gesichtet, keine Kontrolle beobachtet“. Das verfällt nach 3 Halten ohne weitere Meldung. Die Haltepunkt-Abfrage fragt zusätzlich „Kontrollieren sie?“ [ja] [nein].
  2. **`BESTÄTIGT` trägt `aktiv` mit.** Eine Bestätigung von „nur gesehen“ macht nicht 🔴.
  3. **Schutz der Person (Q6):** Meldungen mit `aktiv = false` werden **nicht** zu Teams verknüpft, lösen keine Vorhersage aus, gehen nicht in Statistiken ein, und ein `AUSGESTIEGEN_IN` dazu färbt **keine Station**. So entsteht keine Spur „Kontrolleur fährt abends nach Bergedorf“. Keine Fotos und kein Freitext gelten ohnehin (KONZEPT 4.4).
- **Status:** 🟡 teilweise
- **Restrisiko:** Ohne die Angabe „nur gesehen“ bleibt die Fehlwarnung für bis zu 2 Halte 🔴. Ob jemand im Dienst ist, kann kein Melder sicher wissen; die App behauptet es auch nicht, sondern zeigt nur, ob eine Kontrolle beobachtet wurde.

### K-12 Personal, das wie Kontrolleure aussieht
- **Lösung:**
  1. **Zwei Sicherheitsstufen beim Melden (Q3):** Nachtrag-Chip [Kontrolleure, sicher] [unklar]. `sicher = false` zählt 0,3, erreicht allein höchstens 🟠 mit Etikett „unklar“ und verfällt schnell (Zug: 2 Halte, Station: 10 min). Meldet ein anderes Gerät `sicher = true` und `aktiv = true`, wird daraus normal 🔴.
  2. **Bewusst keine Kategorien „Polizei“ oder „Security“.** Die App verfolgt nur Fahrkartenkontrollen. Eine Polizei-Kategorie würde den Zweck verschieben und das rechtliche Risiko und das Store-Risiko erhöhen (KONZEPT 8).
  3. **Kein Abzug für „unklar“:** Ein Gerät verliert keinen Vertrauenswert, wenn eine „unklar“-Meldung nicht bestätigt wird. Ehrliche Unsicherheit soll sich lohnen. Widerlegte „sicher“-Meldungen senken den Wert leicht.
- **Status:** ✅ gelöst
- **Restrisiko:** Es bleiben 🟠-Fehlalarme, vor allem nachts an Kiez-Stationen. Weil sie klar als „unklar“ gekennzeichnet sind und schnell verfallen, stumpfen sie 🔴 nicht ab. Wer sich sicher irrt, erzeugt weiterhin 🔴, bis eine Entwarnung oder der Verfall greift.

---

## Querschnitt

Bilanz: **4 ✅ gelöst** (K-03, K-06, K-07, K-12), **8 🟡 teilweise** (K-01, K-02, K-04, K-05, K-08, K-09, K-10, K-11), **0 ❌ vollständig nicht lösbar**. Nicht lösbare Teilaspekte: Warnung vor der ersten Erkennung (K-01), Warnung im Moment des gemeinsamen Ausstiegs (K-04), unbeobachtete Einheiten (K-02, K-08) und Gegner mit echten Altgeräten (K-09).

### Q1 Team und Bereich als abgeleitete Objekte (→ KONZEPT 3, 5.1, 6.1, 7)
Löst K-02, K-03, K-05, K-06, K-07, K-08, K-10.
- **Rohmeldungen bleiben dumm** (Tabelle `meldungen`). **Team** und **Bereich** werden auf jedem Gerät deterministisch aus den Meldungen der letzten 2 h berechnet (Sortierung nach `erstellt`, `id`; versionierter Algorithmus). Kein Server-Zustand, nichts davon wird gespeichert.
- **Team** = Hypothese „eine Gruppe bewegt sich“. Zustand: `IN_FAHRT(physischer Zug, einheiten, seit)` | `AM_BAHNSTEIG(station, ausgang?)` | `UNBEKANNT(letzter Ort, seit)`.
- **Zuordnung** einer neuen Meldung `m` (nur `aktiv != false`): Für jedes aktive Team prüft `erreichbar(team, m)` mit Netzgraph und Fahrplan:
  - gleicher physischer Zug (inkl. Flügelpartner) → Wert 1,0
  - Station, die der Zug des Teams seit der letzten Meldung passiert hat → 0,7
  - Fahrt, die an einem passierten Halt ≥ 2 min nach Ankunft abfuhr und `m` plausibel erreicht → 0,5
  - von `AM_BAHNSTEIG`: Fahrt ab dieser Station nach `seit` → 0,7; Nachbarstation in ≤ 2 Kanten → 0,4
  - Wert × `exp(-Δt / 20 min)`, Δt > 45 min → 0.
  Zuordnen, wenn bester Wert ≥ 0,3 **und** ≥ 2 × zweitbester; sonst neues Team. Ordnet `m` ein Team neu zu, endet sein alter Ort nach 1 Halt („vermutlich umgestiegen“).
- **Vorhersage** für Teams im Zustand `UNBEKANNT` oder mit unbestätigter Meldung älter als 2 Halte. Kandidaten mit Startgewichten (⚠️ geschätzt, später aus aggregierten Daten kalibrieren): am Bahnsteig bleiben 0,35 · nächster Zug Gegenrichtung 0,25 · andere Linie am Umstiegsknoten 0,25 (aufgeteilt) · nächster Zug gleiche Richtung 0,15. Angezeigt werden höchstens 3 Kandidaten mit Gewicht ≥ 0,2 als ◌ „möglich“, 20 min lang, nie 🔴.
- **Bereich** `{typ: PENDEL | GROSSLAGE, stationen, kanten, seit, bis_mindestens, n_meldungen}`. Regeln wie in K-03 und K-06.
- **Datenskript (6.1)** erzeugt zusätzlich: linienübergreifenden gerichteten Netzgraphen, Umstiegsknoten (inkl. U-Bahn aus demselben GTFS), Wendefahrten an Endbahnhöfen, `einheiten_erwartet` je Fahrt.

### Q2 Statusmodell: Halte statt Minuten, Entwarnung mit Nachlauf, neue Stufen (→ KONZEPT 5.2, 4.2, 4.5)
Löst K-01, K-04, K-07, K-09, K-10, K-12.
- **Stufen:** 🔴 gemeldet · 🟠 unsicher / angeblich ausgestiegen / nur gesehen / unklar · ◌ möglich (Vorhersage, gestrichelter Ring) · ⚪ beobachtet, nichts gemeldet · ▒ unbeobachtet. Grün gibt es weiterhin nicht.
- **Zugmeldung:** 🔴 bis 2 Halte nach der letzten Meldung (min. 4 min), dann 🟠 für 3 Halte, dann ⚪. `unklar`: maximal 🟠, 2 Halte.
- **`AUSGESTIEGEN_IN`:** Station 🔴 wie bisher; der Zug wird 2 Halte 🟠 „Nachlauf“, ⚪ nur bei zweitem Gerät oder Bestätigung durch den Melder nach der Abfahrt. Späteres `IM_ZUG` hebt den Ausstieg auf.
- **Fahrtende:** überträgt statt zu löschen (Endbahnhof 🟠 20 min, Wendefahrt ◌ 3 Halte).
- **Haltepunkt-Abfrage** in der Fahrtleiste (4.5) für den Melder am nächsten Halt: [noch da] [hier ausgestiegen] [kontrollieren nicht].
- Rote Ränder heißen „bemerkt in“, nicht „eingestiegen in“. Jede Anzeige nennt „zuletzt bestätigt HH:MM bei Station“.

### Q3 Melden mit Nachtrag, Datenmodell erweitert (→ KONZEPT 4.4, 5.4, 7)
Löst K-01, K-02, K-06, K-08, K-09, K-10, K-11, K-12.
- Der erste Tap sendet sofort (2-Tap-Regel bleibt). Danach erscheinen 10 s lang **optionale Chips**, jeder ein Tap: wo im Zug [vorne|Mitte|hinten] · [kontrollieren | nur gesehen] · [sicher | unklar] · [waren schon vorher da → Station] · an Station: [Ausgang …] · dritter Meldepunkt „Großkontrolle“.
- Nachträge ändern die eigene Meldung bis 3 min nach dem Senden über die Edge Function, ohne das Rate-Limit zu belasten.
```sql
alter table meldungen
  drop column zugteil,
  add column einheit         smallint,              -- 1 = vorderste Einheit in Fahrtrichtung … 3; null = unbekannt
  add column seit_station_id text,                  -- "vermutlich schon seit"
  add column aktiv           boolean,               -- true = kontrollieren, false = nur gesehen, null = k. A.
  add column sicher          boolean not null default true,   -- false = "unklar"
  add column gross           boolean not null default false,  -- Großkontrolle
  add column ausgang_id      text;                  -- GTFS-Entrance oder eigene Liste
```
- Beim Check-in vor Ohlsdorf (S1) und in Langzügen fragt die App einmal nach dem Zugteil; danach trägt jede Meldung die Einheit automatisch.

### Q4 Ehrlich mit „keine Meldung ≠ keine Kontrolle“ (→ KONZEPT 5.2, 4.1, 6.4, 7)
Löst K-01, K-02, K-08, K-10, mittelbar alle.
- **Beobachtungsdichte:** Eingecheckte Geräte senden (opt-in) alle 5 min ein `ANWESEND` mit `fahrt_key` und `einheit`, ohne GPS. Der Server führt nur Zähler `anwesenheit(fahrt_key, einheit, bucket_5min, anzahl)`. Für die Deduplizierung genügt ein gehashter Tages-Token, der nach 30 min gelöscht wird.
- Anzeige: ⚪ „keine Meldung · 3 Nutzer an Bord“ vs. ▒ „unbeobachtet“. Einheiten und Außenäste ohne Beobachter erscheinen schraffiert.
- **Grundrate** aus den bereits geplanten aggregierten Statistiken (Station × Wochentag × Stunde): kleines Etikett „hier oft kontrolliert (Di 16–18 Uhr)“, neutral formuliert.
- Feste Texte: beim ersten Start und in der Legende „Keine Meldung heißt nicht keine Kontrolle. Kontrolleure in Zivil sind erst bei der Kontrolle erkennbar.“

### Q5 Gewichtung gegen Manipulation und für Großlagen (→ KONZEPT 5.3)
Löst K-06, K-09, K-12.
- **Asymmetrie:** Entwarnungen (`NICHT_MEHR_DA`, `AUSGESTIEGEN_IN` für den Zugstatus) zählen 0,5 × Gewicht.
- **Neulings-Deckel:** Geräte < 7 Tage oder < 3 bestätigte Meldungen zählen 0,3; ihre Summe pro Objekt ist auf 1,0 begrenzt.
- **Großlage:** Meldungen in einer laufenden Großlage sind Bestätigungen und werden nie als Duplikat oder Spam verworfen.
- **Transparenz:** Die Detailansicht zeigt die Zusammensetzung („2 Meldungen, 1 Entwarnung, 1 neues Gerät“).
- „unklar“ wird nicht bestraft, widerlegtes „sicher“ leicht.

### Q6 Datenschutz-Leitplanken für das Team-Objekt (→ KONZEPT 7 „Datenmodell“, 8)
Löst K-05, K-09, K-11 und ist Voraussetzung für Q1.
- Ein Team, das über Linien wandert, ist ein Bewegungsprofil einer kleinen, bestimmbaren Gruppe (rund 30 Mitarbeitende im Prüfdienst). Deshalb gilt: Teams werden **nur auf dem Gerät** berechnet, **nie gespeichert oder übertragen**. Die Spur wird höchstens 60 min rückwirkend angezeigt, die Kette reißt nach 45 min ohne Meldung, und es gibt keinen Verlauf und keine Statistik pro Team.
- Nur Meldungen mit `aktiv != false` bilden Teams. „Nur gesehen“-Meldungen färben keine Ausstiegsstation (Schutz von Personal außerhalb des Dienstes).
- Statistiken bleiben aggregiert pro Station und Zeitfenster (wie im Konzept). Vorschlag: Rohmeldungen statt nach 24 h schon nach 6 h löschen, weil die App ohnehin nur die letzten 2 h braucht. Vor dem Launch in die anwaltliche Prüfung aufnehmen.
