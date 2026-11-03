# Szenarien K: Verhalten der Kontrolleure

### K-01 Zivilkontrolleure, erst beim „Fahrausweise bitte“ erkennbar
- **Ablauf:** F1 und F2 in Zivil steigen um 08:03 in Wedel in die S1 ein. Fahrgast A sitzt neben ihnen, meldet aber nichts, weil er sie nicht erkennt. Ab Blankenese (08:15) kontrollieren sie. A meldet `IM_ZUG` erst bei Blankenese.
- **App vs. Wirklichkeit:** Die App zeigt „eingestiegen in Blankenese“. In Wahrheit sind sie seit Wedel im Zug. Wer Rissen oder Sülldorf im Blick hatte, sah nur ⚪.
- **Folge:** Wer zwischen Wedel und Blankenese eingestiegen ist, hat einen grauen Zug gesehen und wird erwischt. Schlimmer ist der Lerneffekt: Die Nutzer fangen an, ⚪ als „sicher“ zu lesen.
- **Schwere:** hoch

### K-02 Team teilt sich auf zwei Zugteile auf (Flügelzug S1)
- **Ablauf:** F1 und F2 steigen um 09:10 am Hbf in den vorderen Teil (Richtung Airport). F3 und F4 steigen in den hinteren Teil (Richtung Poppenbüttel). In Ohlsdorf wird der Zug geteilt. A meldet „vorderer Teil“.
- **App vs. Wirklichkeit:** Die Meldung hängt an der Airport-Fahrt. Die Fahrt nach Poppenbüttel erbt sie nur über `partnerOf` oder gar nicht. Die Angabe vorne/hinten ist optional und wird meist weggelassen.
- **Folge:** B im hinteren Teil sieht nach Ohlsdorf „keine Meldung“ und wird in Hoheneichen kontrolliert.
- **Schwere:** hoch

### K-03 Pendeln zwischen zwei Stationen
- **Ablauf:** F1 und F2 fahren 10:00–12:00 nur zwischen Altona und Holstenstraße hin und her, abwechselnd mit S3 und S5, Richtung und Gegenrichtung.
- **App vs. Wirklichkeit:** Jede Fahrt wird kurz rot und nach `AUSGESTIEGEN_IN` wieder ⚪. Stationsmeldungen verblassen nach 20 Minuten. Das eigentliche Muster („Abschnitt Altona–Holstenstraße ist besetzt“) kann das Modell nicht ausdrücken. Es entstehen Dutzende Einzelmeldungen, wer gerade hinschaut, sieht immer nur eine.
- **Folge:** Wer in diesem Abschnitt fährt, wird mit großer Wahrscheinlichkeit erwischt, obwohl „gerade ⚪“ angezeigt wird.
- **Schwere:** hoch

### K-04 Abgangskontrolle direkt nach dem Ausstieg
- **Ablauf:** F1–F3 fahren in der S3 von Harburg bis Harburg Rathaus mit (11:20). Dort steigen sie aus und stellen sich an die Treppe. A meldet `AUSGESTIEGEN_IN Harburg Rathaus`.
- **App vs. Wirklichkeit:** Der Zug wird ⚪ und die Station rot. Das stimmt. Aber Fahrgäste, die mit demselben Zug ankommen, laufen den Kontrolleuren direkt in die Arme. Sie hätten die Warnung schon im Zug gebraucht, bevor die Kontrolleure ausstiegen. Außerdem gilt die Abgangskontrolle nur für einen Ausgang, rot wird aber die ganze Station.
- **Folge:** Die Mitfahrenden im selben Zug werden erwischt. Wer einen anderen Ausgang nimmt, wird unnötig gewarnt.
- **Schwere:** mittel

### K-05 Wechsel auf eine andere Linie
- **Ablauf:** F1 und F2 kontrollieren um 13:05 in der S1 bis Ohlsdorf und steigen dort in die U1. Oder sie wechseln am Berliner Tor um 13:12 von der S1 in die S2 Richtung Bergedorf. Niemand meldet `AUSGESTIEGEN_IN`.
- **App vs. Wirklichkeit:** Die S1-Fahrt bleibt 15 Minuten rot, danach 🟠. Die S2-Fahrt bleibt ⚪, weil der Umstieg nicht erfasst wird und es keine „Team“-Einheit gibt, die die Linie wechseln könnte.
- **Folge:** Doppelter Schaden. In der S1 wird falsch gewarnt, in der S2 werden Leute unerwartet erwischt.
- **Schwere:** hoch

### K-06 Großkontrolle Hbf / Jungfernstieg
- **Ablauf:** Um 16:00 stehen 8 Teams aus Prüfdienst, Hochbahn-Wache und Polizei an allen Ausgängen von Hbf und Jungfernstieg. Mehrere Teams steigen jeweils für eine Station in S1, S2, S3 und S5 zu. Innerhalb von 10 Minuten kommen 60 Meldungen.
- **App vs. Wirklichkeit:** Die Meldungen sind über einzelne Fahrten verstreut und werden zum Teil als Spam oder Duplikat behandelt. Ein Ereignistyp für „Großlage, Bereich gesperrt“ fehlt. Nach 45 Minuten verschwinden die Stationsmeldungen, obwohl die Aktion zwei Stunden läuft.
- **Folge:** Die Nutzer unterschätzen die Lage. Außerdem werden Server und Missbrauchsfilter belastet.
- **Schwere:** hoch

### K-07 Nur eine Station mitgefahren
- **Ablauf:** F1 und F2 steigen um 07:40 an der Sternschanze in die S5, kontrollieren den Wagen und steigen am Dammtor (07:42) aus. A meldet `IM_ZUG` um 07:41.
- **App vs. Wirklichkeit:** Der Zug bleibt bis 07:56 rot, danach 🟠 bis 08:26, weil niemand `AUSGESTIEGEN_IN` meldet. Das Team steht inzwischen am Dammtor auf dem Bahnsteig, und dort wird nichts angezeigt.
- **Folge:** Die Warnung hängt am falschen Ort. Die Leute meiden einen sauberen Zug und steigen am Dammtor unbewarnt in die Kontrolle.
- **Schwere:** mittel

### K-08 Langzug S3 mit drei Einheiten ohne Durchgang
- **Ablauf:** In der S3 aus Pinneberg (3 × ET 490, die Einheiten sind nicht verbunden) steigt F1 um 08:30 in Elbgaustraße in Einheit 1. F2 steigt in Stellingen in Einheit 3. A in Einheit 2 meldet „im Zug“, nachdem er F1 durchs Fenster gesehen hat.
- **App vs. Wirklichkeit:** Das Modell kennt nur vorne/hinten, und das auch nur für die S1. Der ganze Zug wird rot. Für Einheit 2 stimmt das nicht. In Einheit 3 erwartet niemand eine zweite Gruppe.
- **Folge:** Es gibt zu viele Warnungen, gleichzeitig fehlen Teilinformationen. Wer in Einheit 2 aussteigt, um die Einheit zu wechseln, landet womöglich direkt bei F1.
- **Schwere:** mittel

### K-09 Kontrolleure kennen die App und ändern ihr Muster
- **Ablauf:** Der Prüfdienst liest mit. Variante 1: F1 und F2 steigen um 18:10 in Eidelstedt sichtbar aus, A meldet `AUSGESTIEGEN_IN`, die beiden steigen am nächsten Wagen wieder ein. Variante 2: Ein Team macht absichtlich Falschmeldungen auf der S21, während das echte Team auf der S3 unterwegs ist. Variante 3: Kontrolliert wird erst ab der zweiten Station, damit Meldungen zu spät kommen.
- **App vs. Wirklichkeit:** Die App vertraut `AUSGESTIEGEN_IN` und setzt den Zug auf ⚪. Organisierte Falschmeldungen sehen aus wie Bestätigungen und erhöhen das Vertrauen.
- **Folge:** Die Nutzer werden gezielt in die Falle gelockt, die App verliert Vertrauen. Juristisch kann es heikel werden, wenn der Betreiber die App beobachtet.
- **Schwere:** hoch

### K-10 vhh-Prüfteam im Umland, in Uniform, die niemand kennt
- **Ablauf:** Ein vhh-Prüfteam (F1 und F2) steigt um 14:20 in Thesdorf in die S3 nach Pinneberg. Die Fahrgäste kennen die Westen nicht und halten die beiden für Servicepersonal. Gemeldet wird erst bei der Kontrolle in Pinneberg, dem Endbahnhof.
- **App vs. Wirklichkeit:** Die Meldung kommt am Endpunkt, die Fahrt endet gleich danach und die Meldung erlischt. Dass das Team im Gegenzug weiterkontrolliert, sieht niemand. Auf dem dünn genutzten Außenast (z. B. auch Stade) melden ohnehin kaum Leute.
- **Folge:** Im Umland zeigt die App fast immer ⚪. Die Nutzer fühlen sich sicher, obwohl es dort seit 2026 mehr Kontrollen gibt.
- **Schwere:** mittel

### K-11 Kontrolleure nach Feierabend unterwegs
- **Ablauf:** F1 und F2 fahren um 17:45 privat mit der S2 vom Hbf nach Bergedorf, noch in Dienstjacke oder mit erkennbarem Ausweis. A erkennt F1 und meldet `IM_ZUG`, B bestätigt.
- **App vs. Wirklichkeit:** Der Zug ist für 15 Minuten (bei Bestätigungen länger) bestätigt rot, kontrolliert wird aber nichts.
- **Folge:** Es entsteht eine falsche Warnung. Leute warten auf den nächsten Zug und kommen zu spät. Wenn Melder Fotos oder Personenbeschreibungen nachreichen, gerät eine Privatperson ins Visier.
- **Schwere:** niedrig bis mittel

### K-12 Personal, das wie Kontrolleure aussieht
- **Ablauf:** Um 22:30 laufen DB-Sicherheitsdienst und Bundespolizei auf Streife durch die S1 von der Reeperbahn nach Altona. Am Jungfernstieg stehen Mitarbeitende der DB-Information und Leute von einer Fahrgastzählung mit Tablets. A meldet `IM_ZUG` beziehungsweise `AN_STATION`.
- **App vs. Wirklichkeit:** Es gibt Meldungen ohne jede Kontrolle. Weil es keinen Freitext gibt, kann der Melder auch nicht „vielleicht Security“ angeben, und Unsicherheit lässt sich nicht ausdrücken.
- **Folge:** Es häufen sich Fehlalarme, vor allem nachts und an Kiez-Stationen. Die Nutzer stumpfen gegen rote Meldungen ab.
- **Schwere:** niedrig

### Was sich daraus für das Datenmodell ergibt
- Es fehlen ein Objekt für „Team“ oder „Abschnitt“ (K-03, K-05, K-06) und eine Angabe, wie lange die Kontrolleure schon da waren, bevor die erste Meldung kam (K-01, K-10).
- Die Zugteile müssen feiner unterschieden werden als vorne/hinten, nämlich nach Einheit (K-02, K-08). Der Ort einer Stationsmeldung sollte auf einen Ausgang eingrenzbar sein (K-04).
- `AUSGESTIEGEN_IN` ist leicht zu manipulieren und wird selten gemeldet (K-07, K-09). Die Meldungen erlöschen zu früh, sowohl am Fahrtende als auch nach 45 Minuten (K-06, K-10). Es gibt keine Möglichkeit, Unsicherheit anzugeben (K-11, K-12).
