# Lösungen Z: Zugzuordnung

Stand: 3. November 2026. Grundlage: [`../KONZEPT.md`](../KONZEPT.md), [`../recherche/sbahn_hamburg_recherche.md`](../recherche/sbahn_hamburg_recherche.md) (bei Widersprüchen maßgeblich), [`02_zugzuordnung.md`](02_zugzuordnung.md).

Lesehilfe:
- Die Bausteine **Q1 bis Q5** werden im Abschnitt „Querschnitt“ am Ende genau beschrieben. Die Szenarien verweisen nur darauf.
- Wo ein Szenario „grün“ schreibt, ist ⚪ „keine Meldung“ gemeint. Die App zeigt nie Grün (KONZEPT 5.2).
- Faktenfehler in den Szenarien stehen jeweils als kurze **Korrektur** unter der Überschrift.

---

### Z-01 Verspätung im 5-Minuten-Takt: Meldung landet beim Folgezug
> **Korrektur:** In der HVZ fährt die S1 ab Wedel nur alle 10 min (Recherche §4). Der Zug 5 min hinter „S1 08:12 ab Wedel“ ist deshalb ein **Verstärker Blankenese → Poppenbüttel** (Plan Altona 08:44), keine Fahrt „08:17 ab Wedel“. Bei +6 und +4 min liegen die Züge in Altona 3 min auseinander, nicht 1 min. Die Meldung landet einen Zug **zu spät**, also beim Folgezug.

- **Lösung:**
  1. **Check-in per Ereignis (Q3):** A tippt im Zug auf „Ich bin eingestiegen“ und wählt keinen Zug aus der Fahrplanliste. Ankerzeit ist das Anfahren laut Bewegungssensor, ersatzweise der Zeitpunkt des Tippens. Kandidaten sind alle Fahrten mit Plan-Abfahrt in Altona im Fenster [t − 20 min, t + 3 min]. Sie werden nach |Plan + geschätzte Verspätung − t| bewertet. Dazu gilt die FIFO-Regel (Q4): Eine ältere Fahrt, für die es in Altona noch keinen Durchfahrtsnachweis gibt, hat Vorrang, solange sie höchstens einen Takt überfällig ist. Hier gewinnt also „08:12, +6“.
  2. **Ziel an der Zuganzeige als Hauptmerkmal:** In der HVZ wechseln sich auf dem S1-Stamm zwei Zugarten ab: der Flügelzug ab Wedel mit Ziel „Airport/Poppenbüttel“ und der Verstärker ab Blankenese mit Ziel „Poppenbüttel“. Die Check-in-Karte zeigt deshalb groß das Ziel und darunter klein Plan- und Schätzzeit, zum Beispiel „S1 → Airport/Poppenbüttel · Plan 08:39 · ca. +6“ gegen „S1 → Poppenbüttel · Plan 08:44“. Der Nutzer wählt mit einem Tap.
  3. **Meldung mit Anker (Q2):** Die Meldung speichert Altona, 08:45, nächsten Halt Königstraße und das gesehene Ziel. Jedes Handy berechnet die Position des roten Zuges ab diesem Anker mit den Plan-Fahrzeiten. Die Position stimmt dadurch auch dann, wenn der `fahrtKey` falsch geraten wäre. B sieht am Hbf: „🔴 Ankunft ca. 08:58 (gesehen in Altona um 08:45)“.
  4. **Folgezüge (Q4):** Die Durchfahrtszeiten des Melders liefern eine Crowd-Verspätung. Nach der FIFO-Regel kommt der Folgezug frühestens Anker + Mindestzugfolge (ca. 3 min). B sieht beide Züge in der richtigen Reihenfolge.
  5. **Schattenkandidat:** Der zweite Kandidat bleibt 30 min gespeichert. Spätere Signale können den Check-in automatisch umbuchen (`KORREKTUR`), zum Beispiel der erste GPS-Fix nach dem City-Tunnel oder das Verhalten in Ohlsdorf (Flügelung ja oder nein).
- **Status:** 🟡 teilweise (✅ mit Echtzeitquelle, siehe Q4)
- **Restrisiko:** Haben beide Züge dasselbe Ziel und sind beide verspätet (z. B. tagsüber zwei Wedel-Züge bei einer Störung), ist die Wahl nur eine Schätzung. Liegt die Konfidenz unter 0,8, zeigt die App „S1 ab Altona ca. 08:45 – Zuordnung unsicher“. Der gewählte Zug wird 🔴, der zweite Kandidat 🟠, solange seine Wahrscheinlichkeit über 0,25 liegt. Im City-Tunnel gibt es keine GPS-Durchfahrten, dort rechnet die App nur ab dem Anker weiter.

### Z-02 Zugausfall: Meldung hängt an einem Geisterzug
> **Korrektur:** Laut Recherche §3 wäre die S3 „09:05 ab Pinneberg“ um 09:18 in Eidelstedt, um 09:24 in Diebsteich und um 10:09 in Neugraben (nicht 09:50). Die 09:15 ist um 09:28 in Eidelstedt. C steigt um ca. 09:29 also in die echte 09:15. Die App setzt ihn nur dann in die 09:05, wenn sie diese als 11 min verspätet annimmt.

- **Lösung:**
  1. **Check-in (Q3):** Die Kandidaten sind die 09:05 (11 min überfällig, ohne Durchfahrtsnachweis) und die 09:15 (+1). Ist eine Fahrt mehr als einen Takt überfällig, gilt sie als **ausfallverdächtig** und wird im Score abgewertet. Die App wählt sie nur, wenn Crowd-Durchfahrten oder eine Echtzeitquelle sie belegen. Hier gewinnt also die 09:15.
  2. **Auch eine Fehlzuordnung bleibt harmlos (Q2):** Der rote Punkt wird ab dem Anker (Eidelstedt 09:29) berechnet. Er fährt deshalb dort, wo der echte Zug ist, und endet um ca. 10:19 in Neugraben, nicht zur Planzeit der Geisterfahrt.
  3. **Geisterzug erkennen (Q4):** Wird eine spätere Fahrt an einer Station beobachtet, die frühere aber nicht, dann ist die frühere entweder weiter hinten oder ausgefallen, denn auf demselben Gleis gibt es keine Überholung. In beiden Fällen kommt sie nicht mehr vorher. Die App zeigt sie deshalb gestrichelt und „ohne Prognose“ an und nicht mehr als „in 3 min“ in der Abfahrtstafel.
  4. **Crowd-Signal:** In der Abfahrtstafel gibt es den Button „Zug kam nicht“ (Typ `BETRIEB`, niedriges Gewicht). Ab 2 unabhängigen Meldungen gilt die Fahrt als ausfallverdächtig.
  5. **Echtzeit (Q4):** Die DB Timetables API meldet Ausfälle (`fchg`/`rchg`). Damit ist der Ausfall sicher erkannt.
- **Status:** 🟡 teilweise (✅ mit Echtzeitquelle)
- **Restrisiko:** Ohne Echtzeit und ohne Mitfahrer an Bord lässt sich ein Ausfall nicht von einer großen Verspätung unterscheiden. Die Tafel zeigt dann zwei Züge, obwohl nur einer fährt. Die App kennzeichnet deshalb jeden Punkt: hohl heißt „nur nach Fahrplan“, gefüllt heißt „beobachtet vor X min“.

### Z-03 Kurzwende in Barmbek: Meldung fährt in der App weiter nach Ohlsdorf
- **Lösung:**
  1. **Wende beim Melder erkennen:** Die Session merkt, wenn die Fahrtrichtung umschlägt. Das zeigt sich daran, dass die GPS-Projektion auf die Shape rückwärts läuft (Barmbek liegt oberirdisch) oder dass der nächste Halt Friedrichsberg statt Alte Wöhr ist. Die App fragt dann: „Fährt dein Zug zurück Richtung Wedel? [Ja] [Nein]“. Zusätzlich gibt es in der Fahrtleiste (KONZEPT 4.5) den Button „Zug wendet hier“.
  2. **Bei „Ja“:** Die App sendet das Event `BETRIEB: WENDE` mit Fahrt, Station und Zeit. Für alle Handys endet die ursprüngliche Fahrt dann in Barmbek. Der Rest der Fahrt wird als „entfällt vermutlich“ angezeigt, die Zugmeldung endet dort, und am Airport gibt es keinen Fehlalarm.
  3. **Umbuchen:** Die App bindet die Session an die Gegenfahrt, deren Plan-Abfahrt in Barmbek am besten passt. (Ein gewendeter Zug übernimmt meist eine geplante Gegenfahrt. ⚠️ Das ist eine Annahme, nicht belegt.) Passt keine, legt sie eine **Ad-hoc-Fahrt** an: `S1|ADHOC|<Barmbek-DHID>|<Betriebstag>|11:12|Richtung Hasselbrook`. Deren Position wird ab dem Anker mit den Plan-Fahrzeiten der Gegenrichtung berechnet. Die Meldung zieht per `KORREKTUR` mit um. So sieht E in Wandsbeker Chaussee den Zug rot.
  4. **Echtzeit (Q4):** Die DB Timetables API zeigt die entfallenen Halte dieser Zugnummer (Alte Wöhr … Airport). Damit erkennt die App die Wende auch, wenn kein Melder an Bord ist.
- **Status:** 🟡 teilweise
- **Restrisiko:** Ohne Melder an Bord und ohne Echtzeit bleibt die Wende unsichtbar (❌ für diesen Fall). Eine Ad-hoc-Fahrt trägt das Etikett „ungeplanter Zug, Position geschätzt“.

### Z-04 Flügelung Ohlsdorf: Kontrolleur geht in den falschen Zugteil
> **Korrektur:** Im aktuellen Konzept haben die Airport-Fahrt und die Poppenbüttel-Fahrt **denselben** `fahrtKey` (`S1|Wedel|Datum|08:12`), weil Linie, Startstation und Abfahrt gleich sind. Das `partnerOf`-Modell kann die beiden Teile deshalb gar nicht trennen (behoben in Q1). Kornweg erreicht der Zug planmäßig um 09:18. Die BR 474 ist nur innerhalb einer Einheit begehbar. Den Zugteil wechseln kann man also nur über den Bahnsteig, praktisch nur in Ohlsdorf ⚠️.

- **Lösung:**
  1. **Datenmodell (Q1):** Jede Fahrt bekommt einen eigenen `fahrtKey`, der das Ziel enthält. Beide gehören zu einem **Zugverband** mit der Kopplung `FLUEGELUNG Ohlsdorf {vorne: Airport-Fahrt, hinten: Poppenbüttel-Fahrt}`. Eine Meldung vor Ohlsdorf hängt am Verband, nicht an der Airport-Fahrt.
  2. **Zugteil ergibt sich für Eingecheckte von selbst:** Wer nach Ohlsdorf Richtung Kornweg fährt (Station oder GPS), saß hinten. Seine früheren Meldungen bekommen nachträglich `zugteil = hinten`. Beim Check-in vor Ohlsdorf fragt die App einmal: „Wohin fährst du? [Airport (vorne)] [Poppenbüttel (hinten)] [Ich steige vorher aus]“.
  3. **Vererbung ab Ohlsdorf:**
     - Mit Zugteil: Der gemeldete Teil wird 🔴. Der andere Teil bekommt bis zur ersten Station nach Ohlsdorf (Airport bzw. Kornweg) den schwachen Hinweis 🟠 „Wechsel in Ohlsdorf möglich“ und danach ⚪.
     - Ohne Zugteil: Beide Teile werden 🟠 (wie in KONZEPT 5.4).
  4. **Rückfrage nach der Teilung:** Nach der Abfahrt in Ohlsdorf fragt die App die Eingecheckten in **beiden** Teilen: „Kontrolleure in deinem Zugteil? [Ja] [Nein] [Weiß nicht]“. „Ja“ zählt als `BESTÄTIGT` für diesen Teil, „Nein“ als `NICHT_MEHR_DA`. Sobald ein Fahrgast im Teil mit F1 sitzt, wird dieser Teil also richtig rot.
  5. **Zugteil-Frage beim Melden:** Sie erscheint nur auf Flügelzügen vor Ohlsdorf, mit dem Hilfetext „vorne = vorne in Fahrtrichtung, fährt zum Airport“.
- **Status:** 🟡 teilweise
- **Restrisiko:** Wechselt ein Kontrolleur den Zugteil und niemand sieht es, kann die App das grundsätzlich nicht erkennen (❌ für diesen Fall). Ohne Rückmeldung zeigt die App nach Ohlsdorf „Teil unsicher – Wechsel in Ohlsdorf möglich“.

### Z-05 Vereinigung in Ohlsdorf (Gegenrichtung): Meldung geht verloren
- **Lösung:**
  1. **Kopplung erkennen (Q1):** Die Pipeline erzeugt die Kopplung `VEREINIGUNG Ohlsdorf`. Sie erkennt Airport- und Poppenbüttel-Fahrt daran, dass beide ab Ohlsdorf bis Wedel bzw. Blankenese identische Zeiten haben (Recherche §5: 82 Paare je Richtung).
  2. **Status übertragen:** Ab der Abfahrt in Ohlsdorf gilt für den Verband der stärkere Status beider Teile. Die Meldung von H geht ohne Statusverlust über, ihr Anker bleibt Hoheneichen 14:03. Ab Ohlsdorf zeigt die Karte einen Punkt „S1 → Wedel (vereinigt)“.
  3. **Zugteil:** Die Herkunft bleibt als Hinweis sichtbar („gemeldet im Teil aus Poppenbüttel“). Die App fragt in dieser Richtung aber nicht nach vorne/hinten, weil sich die Reihenfolge der Teile nach der Vereinigung aus den Daten nicht ableiten lässt ⚠️.
  4. **Rückfrage:** Eingecheckte im vereinigten Zug bekommen ab Ohlsdorf die Frage „Kontrolleure im Zug?“.
- **Status:** ✅ gelöst
- **Restrisiko:** Kontrolleure können während der Kuppelzeit in Ohlsdorf aus- oder umsteigen. Das fängt die Rückfrage aus Punkt 4 auf, sonst wird der Zug nach der Regel in 5.2 🟠. Fährt ein Teil wegen Verspätung allein weiter (⚠️ nicht belegt), erkennen das nur Crowd-Daten oder Echtzeit. Dann gilt die Meldung für den Teil, in dem der Anker liegt.

### Z-06 HVZ-Verstärker ohne Flügelung wird als Flügelzug gezeigt
> **Korrektur:** Ohne Angabe des Zugteils erbt nach KONZEPT 5.4 auch der Poppenbüttel-Teil des Wedel-Zuges 🟠. Wellingsbüttel und Poppenbüttel sind also nicht ganz ⚪. Markiert ist aber der falsche, um ca. 2 min versetzte Zug, während der echte Verstärker ⚪ bleibt. Altona ist ein Tunnelbahnhof, dort gibt es kein GPS.

- **Lösung:**
  1. **Ziel ist Pflichtmerkmal (Q3):** Die Karten zeigen „S1 → Poppenbüttel (fährt ohne Teilung)“ gegen „S1 → Airport/Poppenbüttel (wird in Ohlsdorf geteilt)“. Liegen zwei Kandidaten weniger als 3 min auseinander, ist die Zeit nicht trennscharf. Die App fragt dann aktiv: „Was steht am Zug? [Poppenbüttel] [Airport/Poppenbüttel]“.
  2. **Schattenkandidat:** Fährt J oder ein anderer Eingecheckter derselben Zuordnung nach Ohlsdorf zum Airport, ist es sicher der Wedel-Zug. Fährt er nach Poppenbüttel, bleibt die Frage offen. Hilfreich ist dann die FIFO-Reihenfolge (Q4), sobald einer der beiden Züge anderswo beobachtet wurde.
  3. **Ohne Klärung:** Die Meldung gilt für beide Kandidaten, der gewählte wird 🔴, der andere 🟠. Das gilt, solange der andere eine Wahrscheinlichkeit über 0,25 hat. Ab Ohlsdorf greifen die Regeln aus Z-04.
- **Status:** 🟡 teilweise
- **Restrisiko:** Der Nutzer kann die Zielanzeige falsch lesen, und im Tunnel gibt es keinen Gegencheck. Die App zeigt dann „Zuordnung unsicher“ und markiert beide Züge.

### Z-07 S5 in Neugraben: Meldung bricht am GTFS-Fahrtende ab
> **Korrektur:** Der Ablauf stimmt (Harburg +35 entspricht 16:40, Neugraben 16:52). Die S3 endet werktags in Neugraben, fährt aber in den Wochenendnächten bis Stade (Recherche §3). Die Pipeline muss die Prüfung dort genauso machen.

- **Lösung:**
  1. **Teilfahrten verschmelzen:** Die Pipeline (KONZEPT 6.1 Schritt 5) verschmilzt die beiden GTFS-Teilfahrten zu **einer** Fahrt mit **einem** `fahrtKey` (Start Elbgaustraße, Ziel Buxtehude bzw. Stade). Die Kopplung heißt `DURCHBINDUNG` und gilt als sicher. Die Regel dafür:
     - gleiche `route_id`,
     - Ankunft und Abfahrt in Neugraben mit 0 bis 3 min Übergang (⚠️ den Wert an den Daten kalibrieren),
     - genau ein Partner in beiden Richtungen (1:1).
     Eine Fahrt, die in Neugraben endet (S3 werktags), ist nie Partner einer S5.
  2. **Unsichere Fälle:** Das betrifft am 10.11. 4 von 90 Fahrten stadtauswärts. Die Kopplung wird als `unsicher` mit Kandidatenliste gespeichert, die Meldung geht als 🟠 an die Kandidaten weiter. Bleibt ein Eingecheckter über Neugraben hinaus im Zug (er bewegt sich weiter und erreicht Fischbek), bestätigt das die Kopplung live (`BETRIEB: DURCHBINDUNG`), und der Status wird 🔴. Endet die Fahrt tatsächlich in Neugraben, greift die Endstationsregel (Q1).
  3. **Regressionstest in der Pipeline:** Die Zahl der verschmolzenen Paare je Tag wird gegen die Recherche geprüft (86/90 stadtauswärts, 90/90 stadteinwärts). Bei mehr als 5 % Abweichung schlägt der Build fehl.
- **Status:** ✅ gelöst
- **Restrisiko:** Bei den wenigen nicht verschmelzbaren Fahrten und an Bautagen bleibt 🟠 statt 🔴. Die App zeigt dann „Weiterfahrt ab Neugraben nicht sicher“.

### Z-08 S7-Schleife: Richtung und Abschnitt wechseln mitten in der Fahrt
> **Korrektur:** Im GTFS ist die S7 je Richtung eine eigene Fahrt (Recherche §3). KONZEPT 6.1 sagt „ein Streifen pro Richtung löst das automatisch“, die Recherche sagt „reicht nicht“. Die Recherche hat Recht: Die beiden Streifen sind richtig, es fehlt aber die **Verbindung der beiden Fahrten in Altona**. Dort wechselt der Zug die Fahrtrichtung ⚠️. Ob dieselbe Einheit weiterfährt, steht nicht im GTFS, weil es keine `block_id` gibt. In Dammtor fährt die S7 nur Richtung Hbf. Die S7 entfällt voraussichtlich zum Fahrplanwechsel am 13.12.2026 ⚠️.

- **Lösung:**
  1. **Kopplung in Altona:** Die Pipeline behält die zwei Streifen und erzeugt die Kopplung `WENDE Altona (unsicher)`. Jede Ankunft aus dem City-Tunnel wird der nächsten S7-Abfahrt Richtung Holstenstraße nach der Mindestwendezeit zugeordnet (häufigster Abstand in den Daten), 1:1 in zeitlicher Reihenfolge.
  2. **Vererbung bei `WENDE`:** Die Meldung geht nur als 🟠 weiter, denn die Kontrolleure können in Altona auch aussteigen. 🔴 wird sie nur, wenn ein Eingecheckter an Bord bleibt, also innerhalb der Wendezeit Richtung Holstenstraße fährt, ohne dass „gehend“ erkannt wird. Dann bestätigt der Server die Kopplung für diese Fahrt.
  3. **Darstellung:** Die Zugkarte zeigt die Schleife als einen Zug mit Abschnittswechsel („S7 → Altona, weiter → Aumühle über Dammtor“). In der Linienübersicht sind die Punkte beider Streifen verknüpft.
  4. **Verwechslung S5/S7 in Dammtor:** Q3 hilft über das Ziel (S7 → Aumühle/Bergedorf, S5 → Neugraben/Stade, S2 → Bergedorf). Spätestens nach dem Hbf korrigiert die App automatisch, denn dort trennen sich die Linien: S7/S2 fahren über Berliner Tor, S5 über Hammerbrook.
- **Status:** 🟡 teilweise
- **Restrisiko:** Ob die Einheit in Altona weiterfährt, ist ohne Mitfahrer nicht sicher, die App zeigt dann 🟠 statt 🔴. Die Sonderlogik wird mit dem Wegfall der S7 hinfällig. Die allgemeine `WENDE`-Kopplung gilt aber für alle Endstationen weiter (Q1).

### Z-09 City-Tunnel: drei Linien, falsche Linie gemeldet
- **Lösung:**
  1. **Kandidaten aller Linien (Q3):** Die Kandidaten in Landungsbrücken kommen aus allen Linien, nicht nur aus der vorgewählten. Die Karten zeigen das Ziel („S1 → Wedel/Blankenese“, „S3 → Pinneberg“, „S7 → Altona“) und dazu den Fahrzeughinweis aus der Recherche §5: Die S3 fährt mit der BR 490 (neuere, durchgehend begehbare Züge, werktags 5–19 Uhr als Langzug mit 9 Wagen), S1 und S7 fahren mit der BR 474 ⚠️ (der Einsatz kann abweichen).
  2. **Schattenkandidaten:** Liegt die Konfidenz unter 0,9, behält die App S3 und S7 als Schattenkandidaten. In Altona trennen sich die Wege: Die S1 fährt nach Bahrenfeld, die S3 nach Diebsteich, die S7 endet. Zeigt der erste GPS-Fix nach dem Tunnel Richtung Diebsteich, bucht die App automatisch um und sendet `KORREKTUR` für die Meldung. Die S3 wird dann rückwirkend ab dem Anker Landungsbrücken rot, die S1 wird neutral. Der Nutzer sieht den Hinweis „Du bist offenbar in der S3 → Pinneberg. [Rückgängig]“.
  3. **Reihenfolge im Tunnel:** Im Tunnel bewerten Durchfahrten und die FIFO-Regel (Q4) alle drei Linien gemeinsam, denn alle teilen sich dieselben Gleise.
  4. **Bis zur Klärung:** Der zweite Kandidat wird 🟠, wenn seine Wahrscheinlichkeit über 0,25 liegt.
- **Status:** 🟡 teilweise
- **Restrisiko:** Zwischen Landungsbrücken und Altona (ca. 6 min) bleibt ein falsch gewählter Zug falsch markiert, denn im Tunnel gibt es kein GPS als Gegenprobe. Die App zeigt bei niedriger Konfidenz „Linie unsicher“.

### Z-10 Zwei Züge gleichzeitig am Bahnsteig (Hbf)
> **Korrektur:** S3 und S5 fahren ab Hbf bis **Neugraben** dieselbe Strecke, nicht nur bis Harburg. Neuwiedenthal wird also richtig gewarnt, erst ab Fischbek fährt nur die S5. Außerdem kennt das Konzept keine Meldung „vom Bahnsteig gesehen: Kontrolleure steigen in Zug X ein“, denn `IM_ZUG` setzt einen Check-in voraus (KONZEPT 5.3).

- **Lösung:**
  1. **Neuer Ereignistyp `EINGESTIEGEN_IN` (Q2):** Der Melder steht an der Station (Nähe-Check) und wählt einen der Züge, die dort laut Plan und Verspätung im Fenster [t − 3, t + 3] min abfahren, über alle Linien und Richtungen. Jede Karte zeigt Linie, Ziel und Gleis (falls das hvv-GTFS Bahnsteig-`stop_id`s hat ⚠️ prüfen) sowie einen Längenhinweis (S3 werktags als Langzug). Mit „Weiß nicht, welcher“ werden beide Kandidaten 🟠.
  2. **Gewichtung:** Wählt der Melder einen Zug eindeutig aus, gilt dieselbe Schwelle wie für eine Meldung ohne Check-in (KONZEPT 11.4). Ohne eindeutige Wahl bleibt es bei 🟠.
  3. **Rückfrage:** Eingecheckte in beiden Zügen werden gefragt „Kontrolleure in deinem Zug?“ (Z-04, Punkt 4). Das klärt die Zuordnung.
  4. **Stationshinweis:** Zusätzlich bekommt die Station Hbf einen schwachen `AN_STATION`-Hinweis, denn die Kontrolleure waren am Bahnsteig.
- **Status:** 🟡 teilweise
- **Restrisiko:** Wählt der Melder den falschen Zug, gibt es ohne Mitfahrer keine Gegenprobe. Ab Fischbek kann die Warnung dann fehlen. Die App zeigt „gesehen vom Bahnsteig, Zuordnung unsicher“.

### Z-11 GPS im Tunnel: Station falsch zugeordnet
> **Korrektur:** Im City-Tunnel (Altona–Hbf) gibt es praktisch kein GPS (Recherche §7). Jungfernstieg und Stadthausbrücke liegen eine Fahrminute auseinander. Der Nähe-Check mit 1,5 km (KONZEPT 5.3) kann die beiden nicht unterscheiden.

- **Lösung:**
  1. **GPS nur bei gutem Fix:** Für den Stationsvorschlag nutzt die App GPS nur, wenn die Genauigkeit höchstens 150 m beträgt und der Fix höchstens 60 s alt ist. Sonst macht sie keinen automatischen Vorschlag.
  2. **Ersatzquellen in dieser Reihenfolge:**
     - Bei Check-in: die Station aus der Anker-Projektion des eigenen Zuges bzw. die Ausstiegsstation (`MELDER_RAUS`).
     - Sonst: zuletzt bekannte Station + vergangene Zeit + Linie. Daraus ergeben sich 2 bis 3 plausible Stationen auf dem Streifen.
     - Sonst: die Favoriten.
  3. **Tunnelstationen:** Die Pipeline markiert Tunnelstationen (City-Tunnel, Harburger Tunnel). Dort muss der Nutzer die Station **immer** bestätigen, mit 2 bis 3 großen Buttons in Streckenreihenfolge. Die App setzt die Station nie stillschweigend.
  4. **Gewicht:** Der Nähe-Check rechnet mit dem Genauigkeitsradius. Meldungen mit schwachem Ortsnachweis bekommen das Gewicht 0,5.
  5. **Korrektur:** Der Melder kann die Station 5 min lang korrigieren (`KORREKTUR`).
- **Status:** ✅ gelöst
- **Restrisiko:** Der Nutzer kann sich vertippen. Stationsmeldungen an Tunnelstationen tragen den Vermerk „Station vom Melder gewählt“. Bestätigungen und Entwarnungen anderer korrigieren Fehler über die Zeit.

### Z-12 Vergessener Check-out: Ein Geistermelder bestätigt falsche Züge
- **Lösung:**
  1. **Lebenszeichen (Q3):** Eine Session gilt nur dann als *aktiv*, wenn in den letzten 10 min mindestens eins davon passiert ist:
     - ein GPS-Fix höchstens 300 m von der Shape und höchstens eine Station von der berechneten Zugposition entfernt,
     - ein passendes Bewegungsmuster „im Fahrzeug“ mit Halten zur Fahrplanzeit,
     - eine Stationsbestätigung in der Fahrtleiste.
     Sonst ist die Session *ruhend*: Sie bekommt keine Rückfragen, und ihre Bestätigungen zählen nicht.
  2. **Automatischer Check-out:** Die App checkt aus und sendet `MELDER_RAUS` mit der letzten plausiblen Station, wenn einer dieser Fälle eintritt:
     - die Fahrt ist an der Endstation,
     - „gehend“ seit mindestens 60 s,
     - GPS mehr als 1 km von der Shape entfernt,
     - der Nutzer steht still, während der Zug laut Berechnung schon mehr als eine Station weiter ist,
     - 45 min ohne Lebenszeichen.
  3. **Fragen ohne Ja-Falle:** Zuerst fragt die App „Bist du noch in der S1 → Airport? Nächster Halt: Barmbek [Ja] [Nein, ausgestiegen in …]“. Erst danach kommt die Frage nach den Kontrolleuren, mit drei gleichwertigen Antworten [Sehe sie] [Sehe sie nicht mehr] [Weiß nicht]. Es gibt keine vorausgewählte Antwort und keinen „Ja“-Button in der Benachrichtigung.
  4. **Keine Crowd-Daten aus ruhenden Sessions:** Eine Durchfahrt zählt nur, wenn ein GPS-Fix an einer Station der Fahrt im erwarteten Zeitfenster (± 10 min) liegt. U-Bahn-Positionen fallen damit heraus.
  5. **Abklingendes Gewicht:** Das Gewicht von `BESTÄTIGT` sinkt mit der Zeit seit dem letzten Lebenszeichen.
- **Status:** ✅ gelöst
- **Restrisiko:** Ohne Freigabe der Bewegungserkennung und im Tunnel gibt es keine automatische Erkennung. Dann bleiben nur das Zeitlimit und die Rückfragen. Die Zugkarte zeigt „zuletzt bestätigt vor X min von 1 Fahrgast“.

### Z-13 Ersatzverkehr wegen Bauarbeiten: Die App zeigt Züge, die nicht fahren
> **Korrektur:** Das hvv-GTFS enthält zumindest einen Teil der Bauarbeiten (Recherche: Sa 14.11. weicht ab), aber nur so, wie sie beim Erstellen des Datensatzes bekannt waren. Ob SEV-Busse im GTFS als eigene Route stehen, ist ungeklärt ⚠️.

- **Lösung:**
  1. **Datensatz täglich prüfen (Q5),** nicht nur monatlich, damit nachgetragene Bauänderungen ankommen.
  2. **SEV aus dem GTFS:** Steht der SEV im GTFS (Bus-`route_type` 3/700–716 mit S-Bahn-Bezug ⚠️ prüfen), zeigt die App den Abschnitt auf dem Streifen gestrichelt als „Ersatzverkehr“. Dort gibt es keine Zugkarten und keinen Check-in.
  3. **Ohne Daten, Crowd-Signal:** Mit `BETRIEB: KEIN_ZUG` können Nutzer einen Abschnitt melden (Station von–bis). Ab 2 unabhängigen Meldungen oder mit Bestätigung durch eine Echtzeitquelle wird der Abschnitt grau („kein Zugverkehr gemeldet“). Check-ins in Fahrten dieses Abschnitts sind dann gesperrt, bestehende Check-ins bekommen eine Rückfrage. Fahrten in diesem Abschnitt gelten als ausfallverdächtig (Z-02).
  4. **Echtzeit (Q4):** Ausfälle an DB-Stationen über die DB Timetables API, ServiceAlerts über gtfs.de, falls Hamburg dort enthalten ist.
  5. **Meldungen im SEV:** Im SEV gibt es nur Stationsmeldungen mit dem Ort „Ersatzhaltestelle“ (`ort = 'SEV'`). Sie färben die Station anders als eine Bahnsteigmeldung. Meldungen zu einem bestimmten Bus sind nicht vorgesehen.
- **Status:** 🟡 teilweise
- **Restrisiko:** Ein kurzfristiger SEV ohne Daten und ohne Meldungen bleibt unsichtbar. Einzelne Busfahrten lassen sich nicht abbilden (❌ für diesen Teil). Solange die App keine Echtzeit hat, zeigt sie dauerhaft den Hinweis „Fahrplan ohne Echtzeit – Bauarbeiten möglich“.

### Z-14 Fahrplanwechsel Dezember 2026: S7 entfällt, S2 fährt wieder nach Altona
> **Korrektur:** Der aktuelle Datensatz gilt nur bis 12.12.2026. Ein JSON daraus enthält für den 14.12. **keine** Fahrten. Ein altes Handy zeigt also keine S7, sondern gar nichts, es sei denn, die App rechnet Wochentage einfach fort. Das darf sie nicht. Das Problem „zwei Datenwelten“ ist aber echt, und zwar bei **jedem** Datensatzwechsel, nicht nur im Dezember. Die Umstellung S7/S2 ist laut Recherche wahrscheinlich, aber nicht fest ⚠️.

- **Lösung (Q5):**
  1. **Neuer Datensatz:** Die Pipeline prüft täglich, ob es einen neuen Datensatz gibt. Sie veröffentlicht ihn **vorab** mit Gültigkeitsfenster. Die App lädt den nächsten Datensatz, sobald er da ist, und aktiviert ihn ab seinem ersten Betriebstag.
  2. **Kein Fortrechnen:** Deckt kein Datensatz den Betriebstag ab, schaltet die App auf „nur Stationsmeldungen“ mit Banner. Sie rechnet keinen Fahrplan fort.
  3. **Stabiler `fahrtKey` (Q1):** Der Schlüssel besteht nur aus Feldern, die über Datensätze hinweg stabil sind (DHID statt Name, Betriebstag, GTFS-Zeit). Bleibt eine Fahrt unverändert, ist ihr Schlüssel in jeder Version gleich.
  4. **Version in jeder Meldung:** Jede Meldung trägt `datensatz` und den Anker (Q2). Findet ein Handy den `fahrtKey` in seinem eigenen Datensatz nicht, ordnet es die Meldung über den Anker zu: Station, Zeit und nächster Halt, also der Zug, der dort zu dieser Zeit in diese Richtung fuhr. So landet eine Meldung „S7 07:48“ von einem alten Handy auf einem neuen Handy bei der S2, die tatsächlich dort fuhr.
  5. **Serverprüfung:** Der Server lehnt Meldungen ab, deren Datensatz den Betriebstag nicht abdeckt, und antwortet „bitte Fahrplan aktualisieren“.
  6. **Netzänderung:** Linien sind nie hart codiert (KONZEPT 6.1). Die Sonderlogik der S7 (Z-08) entfällt automatisch, wenn die Daten keine S7 mehr enthalten.
- **Status:** ✅ gelöst
- **Restrisiko:** Veröffentlicht der hvv den neuen Datensatz zu spät, gibt es ab dem Wechseltag keine Zugpositionen. Stationsmeldungen (DHID) funktionieren aber unabhängig vom Datensatz weiter. Die App zeigt „Neuer Fahrplan noch nicht verfügbar“.

### Z-15 Kontrolleure steigen um, Meldung bleibt am alten Zug
> **Korrektur:** Nach KONZEPT 5.2 wird die S1 15 min nach der letzten Bestätigung 🟠 und nach weiteren 30 min ⚪. „Rot bis Ohlsdorf“ stimmt also nicht, 🟠 bis etwa Barmbek aber schon.

- **Lösung:**
  1. **Umstiegs-Heuristik (läuft deterministisch auf jedem Handy):** Sie greift, wenn alle drei Bedingungen erfüllt sind:
     - Eine neue Zugmeldung M2 betrifft Fahrt B, deren Weg rückwärts eine Station S kreuzt, an der die gemeldete Fahrt A war (hier Altona).
     - A und B waren in S höchstens 8 min auseinander.
     - M2 kommt höchstens 15 min nach der letzten Bestätigung von M1.
     Dann gilt „wahrscheinlicher Umstieg in S“. A wird ab S 🟠, ihr Vertrauen halbiert sich. S bekommt für 10 min einen 🟠-Stationshinweis. B wird rückwirkend ab S als „vermutlich seit Altona“ markiert. Kommen mehrere Stationen in Frage, nimmt die App die späteste.
  2. **Gezielte Rückfrage:** Eingecheckte in A, die nach S noch mitfahren, bekommen die Frage „Kontrolleure noch im Zug?“ (Regeln aus Z-12).
  3. **Anzeige und Statistik:** Die App fasst die beiden Meldungen als „vermutlich dieselbe Kontrolle“ zusammen und zählt sie nicht als zwei Teams.
  4. **Ausstieg melden:** Den Button „Sie sind ausgestiegen“ in der Fahrtleiste (KONZEPT 4.5) gibt es schon. Nach einer eigenen `IM_ZUG`-Meldung wird er hervorgehoben.
- **Status:** 🟡 teilweise
- **Restrisiko:** Der Prüfdienst arbeitet mit rund 30 Personen in mehreren Teams (Recherche §8). Die Heuristik kann deshalb zwei echte Teams fälschlich zusammenlegen. Sie setzt den Zug darum nur auf 🟠 und löscht nie etwas. Zwischen Altona und Diebsteich bleibt ohne Beobachter grundsätzlich eine blinde Lücke.

---

## Querschnitt

Fünf Änderungen am Konzept lösen die meisten Szenarien gemeinsam.

### Q1 Zug, Fahrt und Kopplung trennen, neuer `fahrtKey` (KONZEPT §3 „Stabile Fahrt-ID“, §5.4, §6.1 Schritt 5)
Löst ganz oder teilweise: Z-04, Z-05, Z-06, Z-07, Z-08, Z-14.

**Neuer Schlüssel:**
```
fahrtKey = "<Linie>|<Start-DHID>|<Ziel-DHID>|<Betriebstag>|<Abfahrt am Start als GTFS-Zeit>"
Beispiel:  "S1|de:02000:<Wedel>|de:02000:<Airport>|2026-11-10|08:12"
```
- **Ziel im Schlüssel:** Ohne das Ziel haben die Airport- und die Poppenbüttel-Fahrt eines Flügelpaares denselben Schlüssel (Fehler im heutigen Konzept, siehe Z-04).
- **DHID statt Name:** Die DHID ist bundesweit stabil, Stationsnamen können sich ändern.
- **Betriebstag und GTFS-Zeit:** Die Zeit kann über 24:00 liegen. So bleiben Nachtfahrten nach Mitternacht beim richtigen Betriebstag.

**Kopplungen im JSON:**
```
kopplungen: [{ typ, station, von: [fahrtKey], nach: [fahrtKey], zugteil?: {vorne, hinten}, sicher: bool }]
```
Ein **Zugverband** ist eine Kette von Fahrten, die über Kopplungen verbunden sind. Er ersetzt `partnerOf`. Meldungen hängen am Verband, die Vererbung folgt den Kanten:

| Kopplung | Herkunft | Vererbung der Meldung |
|---|---|---|
| `FLUEGELUNG` Ohlsdorf | GTFS: identische Zeiten bis Ohlsdorf | mit Zugteil: Teil 🔴, anderer Teil kurz 🟠; ohne Zugteil: beide 🟠 (Z-04) |
| `VEREINIGUNG` Ohlsdorf | GTFS: identische Zeiten ab Ohlsdorf | stärkerer Status beider Teile (Z-05) |
| `DURCHBINDUNG` (S5 Neugraben, auch S3 nachts) | Pipeline, 1:1, 0–3 min Übergang | voll; bei unsicherer Kopplung 🟠 (Z-07) |
| `WENDE` (S7 Altona, alle Endstationen) | Pipeline-Heuristik, unsicher (keine `block_id`) | 🟠; 🔴 nur, wenn ein Eingecheckter an Bord bleibt (Z-08) |

**Endstationsregel:** Endet eine Fahrt mit aktiver Zugmeldung ohne sichere Kopplung, bekommt die Endstation für 10 min einen schwachen 🟠-Hinweis „Kontrolleure vermutlich hier ausgestiegen“.

### Q2 Meldung als Beobachtung mit Anker, Zuordnung korrigierbar (KONZEPT §5.1, §6.2, §7 Datenmodell)
Löst ganz oder teilweise: Z-01, Z-02, Z-03, Z-09, Z-10, Z-14, Z-15.

Eine Meldung beschreibt zuerst, **was wo wann gesehen wurde**. Der `fahrtKey` ist nur die beste Schätzung dazu und kann später korrigiert werden.

```sql
alter table meldungen
  add column anker_station  text,         -- DHID, wo beobachtet
  add column anker_zeit     timestamptz,  -- Zeitpunkt des Ereignisses (Abfahrt/Einstieg), nicht Sendezeit
  add column anker_naechste text,         -- DHID des nächsten Halts (= Richtung, unabhängig von der Linie)
  add column linie_gesehen  text,
  add column ziel_gesehen   text,         -- was der Nutzer an der Zuganzeige bestätigt hat
  add column kandidaten     jsonb,        -- [{fahrt_key, p}] bei unsicherer Zuordnung
  add column datensatz      text,         -- Fahrplanversion auf dem Gerät
  add column bezug_id       uuid;         -- für KORREKTUR / BESTÄTIGT
```
**Neue Ereignistypen:**
- `KORREKTUR`: hängt eigene frühere Meldungen auf eine andere Fahrt, Station oder einen anderen Zugteil um. Der Server prüft, dass sie vom selben Gerät kommt.
- `EINGESTIEGEN_IN`: vom Bahnsteig gesehen, die Kontrolleure steigen in einen bestimmten Zug ein.
- `DURCHFAHRT`: Crowd-Zeitstempel ohne Kontrollbezug (Q4).
- `BETRIEB`: `WENDE`, `KEIN_ZUG`, `DURCHBINDUNG` bestätigt.

**Position gemeldeter Züge:** Sie wird ab dem jüngsten Anker berechnet: `positionVon(fahrt, jetzt, verspaetung = anker_zeit − plan(fahrt, anker_station))`. Der gemeldete Zug ist damit immer der am besten verortete, auch ohne Echtzeit und auch bei falschem `fahrtKey`.

**Darstellung:** Ein hohler Punkt heißt „nach Fahrplan“, ein gefüllter Punkt heißt „beobachtet vor X min“.

### Q3 Check-in per Ereignis statt per Fahrplan, mit Schattenkandidaten und Lebenszeichen (KONZEPT §6.3, §6.5, §4.4, §4.5)
Löst ganz oder teilweise: Z-01, Z-02, Z-06, Z-09, Z-10, Z-11, Z-12.

1. **Auslöser:** Der Nutzer tippt im Zug auf „Ich bin eingestiegen“. Ein Zug wird nicht aus der Fahrplanliste gewählt. Die Station kommt aus GPS (höchstens 150 m Genauigkeit, Fix höchstens 60 s alt), sonst aus Favorit oder letzter Station, und der Nutzer bestätigt sie.
2. **Ereigniszeit:** Mit Bewegungserkennung ist es das Anfahren innerhalb von 3 min nach dem Tippen, sonst der Zeitpunkt des Tippens.
3. **Kandidaten:** alle Linien und Fahrten mit Plan-Abfahrt im Fenster [t − 20, t + 3] min an dieser Station. Score:

   `p ∝ exp(−((t − (plan + δ̂)) / σ)²) × FIFO × Ausfallmalus`

   - δ̂ ist die geschätzte Verspätung aus Q4.
   - Der FIFO-Faktor schließt Fahrten mit Durchfahrtsnachweis aus und bevorzugt die älteste Fahrt ohne Nachweis.
   - Der Ausfallmalus trifft Fahrten, die mehr als einen Takt überfällig sind.
4. **Auswahl-UI:** 2 bis 3 große Karten. Oben steht das **Ziel laut Zuganzeige**, darunter Plan- und Schätzzeit, dazu Fahrzeug- oder Längenhinweise. Liegen zwei Kandidaten weniger als 3 min auseinander, fragt die App aktiv „Was steht am Zug?“.
5. **Schattenkandidaten:** Sie bleiben 30 min gespeichert. Jedes neue Signal aktualisiert die Wahrscheinlichkeiten: GPS auf der Shape mit der jeweils besten Verspätung, Verzweigungen, Halte, Crowd-Daten. Überschreitet ein anderer Kandidat 0,9, bucht die App automatisch um, sendet `KORREKTUR` und bietet „Rückgängig“ an.
6. **Lebenszeichen und automatischer Check-out:** wie in Z-12. Nur aktive Sessions bekommen Rückfragen, liefern Durchfahrten und tragen Gewicht. In der Fahrtleiste kann der Nutzer auf „jetzt: Königstraße“ tippen und die Station korrigieren. Das ist zugleich ein manueller Anker im Tunnel.

### Q4 Crowd-Durchfahrten mit FIFO, Echtzeit früher einplanen (KONZEPT §2 Tabelle, §6.4, §7, §10)
Löst ganz oder teilweise: Z-01, Z-02, Z-03, Z-09, Z-13.

**Durchfahrten:** Aktive Sessions senden `DURCHFAHRT` (Fahrt oder Kandidaten, Station, Ist-Zeit) mit einer zufälligen ID pro Check-in statt der Geräte-ID. Der Server löscht sie nach 3 h. Die Handys berechnen daraus selbst die Verspätung je Fahrt (Median der jüngsten Beobachtungen, stromabwärts konstant fortgeschrieben). Das Backend bleibt dabei „dumm“ (KONZEPT §7).

**FIFO auf gemeinsamen Gleisen:** City-Tunnel, Verbindungsbahn, Harburger S-Bahn und S1-Stamm. Es gilt: Prognose(Fahrt n) ≥ Prognose(Fahrt n−1) + Mindestzugfolge (ca. 3 min, Recherche §4). Wo der Sollfahrplan selbst die Reihenfolge ändert, liegt eine Überholung vor. Die Pipeline erkennt diese Stellen, und dort setzt die Regel aus ⚠️. Eine Fahrt ohne Beobachtung, deren Nachfolger schon gesehen wurde, ist ausfallverdächtig.

**Echtzeitquelle:**
- **DB Timetables:** Ein kleiner Worker (Supabase Cron oder Mini-VPS) fragt `rchg` für die ca. 70 S-Bahn-Stationen reihum ab und `plan` stündlich. Das sind etwa 50 Aufrufe/min bei einem Limit von 60. Er ordnet die Züge über Linie, Plan-Zeit, Ziel und eine einmalig gepflegte Tabelle DHID ↔ EVA einem `fahrtKey` zu, speichert die DB-Zugnummer und verteilt Verspätungen und Ausfälle über Supabase Realtime.
- **gtfs.de GTFS-RT:** parallel testen, ob die S-Bahn Hamburg enthalten ist ⚠️. Die Zuordnung läuft über DHID + Plan-Zeit, weil der Feed zum gtfs.de-Statikdatensatz passt und nicht zum hvv-GTFS.
- **Geofox GTI:** nicht einplanen.

### Q5 Versionierte Fahrplandatensätze (KONZEPT §6.1, §7, §11 Frage 5)
Löst ganz oder teilweise: Z-13, Z-14.

- **Pipeline:** Sie prüft **täglich**, ob ein neuer Datensatz da ist. Jede Version liegt als `sbahn-hh-<gültigAb>-<hash>.json` vor. Eine `manifest.json` listet alle Versionen mit ihrem Gültigkeitsfenster.
- **App:** Sie hält die aktuelle und die nächste Version und wählt je Betriebstag die passende. Sie rechnet nie über das Gültigkeitsende hinaus. Deckt keine Version den Tag ab, schaltet sie auf „nur Stationsmeldungen“.
- **Meldungen:** Jede Meldung trägt `datensatz`. Passt der `fahrtKey` nicht zur eigenen Version, ordnet das Handy die Meldung über den Anker (Q2) zu. Der Server lehnt Meldungen aus Versionen ab, die den Betriebstag nicht abdecken.

### Reicht der reine Fahrplan ohne Echtzeit-API für Phase 1?

**Ja, für Phase 1 im Sinne von KONZEPT §10** (MVP auf der S1, Testgruppe mit Freunden). Dafür müssen drei Bedingungen erfüllt sein:
1. Check-in per Ereignis (Q3) statt „nächster Zug“.
2. Position gemeldeter Züge ab dem Anker (Q2).
3. Ehrliche Darstellung „nach Fahrplan“ gegen „beobachtet“.

Das reicht, weil jede Zugmeldung ihre eigene Ortung mitbringt. Der wichtigste Zug, nämlich der gemeldete, ist auch ohne API richtig verortet.

**Nicht mehr reicht er, sobald einer dieser Punkte eintritt:**
1. **Öffentlicher Launch** (KONZEPT §9 Punkt 4) oder Ausweitung über die S1 hinaus. Die Abfahrtstafel ist laut §9 Punkt 1 der Nutzen ohne Meldungen. Ohne Verspätungen und Ausfälle ist sie schlechter als die hvv-App, und damit fällt dieses Kaltstart-Argument weg.
2. **Abschnitte mit dichtem Takt.** Im City-Tunnel kommt im Schnitt alle 2,5 min ein Zug, auf dem S1-Stamm in der HVZ etwa alle 5 min, nach Harburg ebenfalls alle 5 min. Dort reicht schon eine Verspätung von einem halben Zugabstand (ca. 1,5 bis 2,5 min), damit nicht gemeldete Züge falsch zugeordnet werden. Das betrifft auch das Matching für Nutzer wie B am Hbf.
3. **Störungen.** Ausfälle, Kurzwenden und SEV (Z-02, Z-03, Z-13) erkennt ohne Mitfahrer nur eine Echtzeitquelle.

**Messbarer Auslöser schon im Test:** Echtzeit wird nötig, wenn in der HVZ mehr als 10 % der Check-ins später eine `KORREKTUR` brauchen oder unter 0,8 Konfidenz bleiben.

**Empfehlung:** Die DB Timetables API mit Cache-Worker von Phase 3 in **Phase 2** vorziehen, also **vor** den öffentlichen Launch. Die Zeile „Verspätungen: optional, später“ in KONZEPT §2 und die Stufe 3 in §6.4 entsprechend ändern.
