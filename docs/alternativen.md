# Alternativen zur Zugzuordnung: robustere Funktionsweisen

Stand: 3. November 2026. Bezieht sich auf [`KONZEPT.md`](KONZEPT.md) Version 2 und [`reviews/codex_review_konzept_v2.md`](reviews/codex_review_konzept_v2.md). KONZEPT.md bleibt unverändert. Unsichere Punkte sind mit ⚠️ markiert. Rechtliche Fragen (vor allem zu Fotos) werden separat recherchiert und hier nicht bewertet.

**Ausgangsfrage:** Der Nutzer befürchtet, dass die App zu unzuverlässig wird. Das Review nennt als Kern: Eine Beobachtung lässt sich im dichten Takt und bei Verspätungen nicht sicher dem richtigen Zug zuordnen. Zwei verspätete Züge mit gleichem Ziel bleiben unter Umständen ununterscheidbar, auch mit Echtzeit.

---

## 1. Warum Konzept v2 an dieser Stelle zerbrechlich ist

Konzept v2 enthält schon die richtige Idee, nämlich die **Meldung als Beobachtung mit Anker** (5.1). Daraus berechnet es die Position des gemeldeten Zuges, auch wenn der `fahrtKey` falsch geraten ist (Lösung Z-01, Z-02). Trotzdem baut v2 alles Weitere auf der Fahrt auf:

| Baustein in v2 | Was er braucht | Was bei falscher Zuordnung passiert |
|---|---|---|
| Zugstatus je Fahrt und Zugteil (5.3) | richtige Fahrt | Falscher Zug wird 🔴, der richtige zeigt ⚪ „keine Meldung · Nutzer an Bord“. Das ist ein **stiller Fehler**: Er sieht aus wie eine gute Nachricht. |
| Check-in mit Zielwahl, Schattenkandidaten (6.3) | Mitwirkung des Nutzers, Echtzeit | Bei zwei Kandidaten unter 3 min Abstand bleibt nur Raten. |
| Datenlage „Nutzer an Bord“ (5.2) | eingecheckte Nutzer in vielen Zügen | Bei geringer Dichte ist fast alles ▒ „keine Daten“. |
| Rückfragen, Crowd-Verspätung, Teams (4.5, 6.4, 5.5) | mehrere Eingecheckte pro Zug | Funktioniert erst bei hoher Nutzerdichte. |
| Echtzeit-Worker (6.4) | DB Timetables, Abgleich EVA ↔ DHID ↔ `fahrtKey` | Neue Fehlerquelle: alte Daten, falsche Zuordnung der Zugnummer. |

Die Zugzuordnung ist also ein **Einzelpunkt, an dem vieles hängt**, und sie braucht genau das, was beim Start fehlt: viele Nutzer und gute Echtzeitdaten.

---

## 2. Die Ansätze im Einzelnen

### 2.1 Unsicherheitsband statt Zug-Zuordnung

**Idee:** Eine Meldung ist ein Punkt: Station A, Zeit t₀, Richtung (nächster Halt), Linie falls bekannt. Daraus läuft ein Band möglicher Positionen auf dem **Gleisgraph** (nicht auf der Fahrt) weiter. Die App prüft, ob ein Zug das Band schneidet.

```
Für jede Station S stromabwärts von A:
  T_erw(S)   = t0 + Fahrzeit(A → S)              (Median aus dem GTFS)
  Breite(S)  = ± (e_meld + e_lauf · Halte(A → S)) Startwerte: e_meld = 1 min
                                                 (0,5 min, wenn der Zug gerade am Bahnsteig stand),
                                                 e_lauf = 0,15 min pro Halt
  Gewicht(S) = p0 · Abklingen pro Halt            (wie v2 5.3)
Ein Zug, der S zur Zeit T_Z erreicht, liegt im Band, wenn |T_Z − T_erw(S)| ≤ Breite(S).
```

**Warum das robust ist:**
- **Verspätungen kürzen sich heraus.** Die Ankerzeit ist eine echte Uhrzeit. Ist der Zug 7 min verspätet, steckt das schon in t₀. Es zählt nur, wie sich die Verspätung zwischen A und S **ändert**, und das ist auf wenigen Halten klein.
- **Kein Echtzeit-API nötig für den wichtigsten Moment.** Wer am Bahnsteig steht, braucht für T_Z nur „jetzt“: „Der Zug, der jetzt einfährt, liegt im Band“ (oder nicht). Echtzeit verbessert nur die Vorschau („in 6 min“).
- **Fehler fallen breit statt falsch aus.** Ist das Band breiter als der Takt, zeigt die App „einer von 2 Zügen“, beide 🟠. v2 würde einen raten und den anderen ⚪ lassen.
- **Band und Abklingen passen zusammen.** Das Band ist dort schmal, wo die Meldung noch stark ist (die nächsten 3–5 Halte), und wird erst breit, wenn die Meldung ohnehin schon zu 🟠/◌ abgeklungen ist.
- **Keine Überholung auf gemeinsamen Gleisen** (Recherche 6.2): Die Reihenfolge der Züge im Band bleibt fest.

**Rechenbeispiel S1 (HVZ, Takt 4/6 min):** Meldung „im Zug Richtung Airport, Blankenese ab 08:21“.
- Altona (6 Halte, +14 min): 08:35 ± 1,9 min. Das Band ist rund 3,8 min breit, also meist **1 Zug**, an der Grenze 2.
- Jungfernstieg (11 Halte, +24 min): 08:45 ± 2,7 min, also 1–2 Züge. Gewicht nach 11 Halten mit Knoten Altona ist unter 0,2, also nur noch ◌ „möglich“.
- Im City-Tunnel (S1+S3+S7, Ø 2,5 min) zählt das Band nur Züge der gemeldeten Linie. Ist die Linie unbekannt (Beobachtung vom Bahnsteig), umfasst es 1–2 Züge aller Linien. An Verzweigungen (Hbf, Ohlsdorf) teilt sich das Band, wenn Linie oder Zugteil fehlen.

**Grenzen:**
- Bei Störungen ändern sich Verspätungen stark. Meldet die Echtzeit eine Störung, wird das Band verbreitert und als „unsicher, Störung“ beschriftet.
- Ein später gesendeter Bericht („waren schon in Rissen da“) hat eine unscharfe Ankerzeit. Lösung: Tippzeit ist Ankerzeit (wie v2), Nachträge vergrößern e_meld.
- Weniger „Wow“ als „jeder Zug einzeln“. Dafür ist die Aussage ehrlich und prüfbar.

**Messbar (wichtig):** Kommt eine zweite, unabhängige Meldung stromabwärts, lag sie im vorhergesagten Band oder nicht? Diese **Band-Trefferquote** ist eine einfache Kennzahl, mit der sich e_meld und e_lauf einstellen lassen. Ziel: ≥ 80 % Treffer bei im Median ≤ 1,5 Zügen im Band auf den ersten 5 Halten.

### 2.2 Abschnitts- und Stationsrisiko statt Zugstatus

**Idee:** Pro Abschnitt zwischen Knoten (z. B. „S1 Blankenese–Altona“, „City-Tunnel Altona–Hbf“, „Hbf–Harburg“) und pro Station eine Anzeige der letzten 20–30 min: „S1 Altona–Hbf Richtung Airport: 2 Meldungen in 20 min, zuletzt 08:41 Reeperbahn“.

- **Stärke:** braucht keine Zuordnung, verträgt wenige Meldungen, passt zum Pendeln von Teams auf 2–3 Stationen (K-03), funktioniert im Web und in Telegram. FreiFahren arbeitet mit einer Risiko-Ampel pro Linie.
- **Schwäche:** beantwortet nicht „welcher Zug?“. Eine Ampel verführt zu „grün = sicher“. Deshalb wie in v2: keine grüne Stufe, nur „keine Meldung“ in Grau, und Meldehäufigkeit ist nicht gleich Kontrollwahrscheinlichkeit (Review 6).
- **Rolle:** Übersicht und Rückfallebene. Die Abschnittsanzeige entsteht automatisch als Zusammenfassung der Bänder (2.1) und Stationsmeldungen (2.4).

### 2.3 Physische Zugidentität ohne Fahrplan

**a) Bluetooth Low Energy mit rotierenden IDs**

| Plattform | Machbarkeit |
|---|---|
| iOS | Werben und Suchen im Vordergrund ja. Im Hintergrund landen Service-UUIDs im „Overflow“-Bereich und werden von anderen iPhones kaum gefunden, besonders bei gesperrtem Bildschirm. Die Corona-Warn-App nutzte dafür das Exposure-Notification-Framework von Apple/Google, das nur Gesundheitsbehörden bekommen. Erfahrung aus TraceTogether (Singapur 2020): Die App musste auf iPhones im Vordergrund offen bleiben. |
| Android | Machbar mit Vordergrunddienst und Daueranzeige, Berechtigungen `BLUETOOTH_SCAN/ADVERTISE` (Android 12+), Scan-Drosselung, Akku-Killer mancher Hersteller. |
| Web | **Nicht machbar.** Web Bluetooth kann nicht werben, freies Suchen ist experimentell (Chrome hinter Flag), Safari hat gar kein Web Bluetooth. |

- **Fehlerquellen:** BLE reicht durch Fenster. An Mittelbahnsteigen (Altona, Jungfernstieg, Hbf) steht oft ein Gegenzug daneben. Zuordnung erst nach gemeinsamer Fahrt über ≥ 2 Halte.
- **Nötige Dichte (Schätzung):** FreiFahren hat in Berlin rund 40.000 regelmäßige Nutzer, aber nur etwa **100 gleichzeitig aktive** im ganzen Netz (Konkurrenz-Recherche). In der Hamburger HVZ fahren rund 66 S-Bahnen. Damit in einem Zug mit 50 % Wahrscheinlichkeit **mindestens 2** aktive Geräte sind, braucht man im Mittel etwa 1,7 pro Zug, also über 110 gleichzeitig aktive Geräte nur in der S-Bahn, und zwar mit laufendem BLE. Das ist mehr, als FreiFahren in ganz Berlin hat.
- **Datenschutz und Sicherheit, entscheidend:** Rotierende IDs verstecken nicht, **dass** jemand die App benutzt. Die Werbepakete tragen eine feste App-Kennung (Service-UUID oder Herstellerdaten). Jeder mit einer kostenlosen BLE-Scanner-App, auch der Prüfdienst, kann im Wagen zählen, wie viele App-Nutzer da sind, und sie über die Signalstärke grob orten. Die Corona-Warn-App war davor geschützt, weil weltweit alle Warn-Apps dieselbe Kennung nutzten. Hier würde die App ihre Nutzer **sichtbar machen**. Das ist ein Ausschlussgrund.

**b) Fahrzeugnummer eintippen oder scannen**

- Jede Einheit der BR 474 und 490 trägt eine Nummer. Ob sie **innen** gut lesbar ist (über Türen, an Wagenenden), ⚠️ vor Ort prüfen.
- **Stärke:** identifiziert die Einheit, also genau den Zugteil (löst Ohlsdorf, Z-04, und Langzug S3). Bleibt über Wenden und Fahrten hinweg gleich.
- **Schwäche:** Wer am Bahnsteig wartet, kennt die Nummer des kommenden Zuges erst, wenn er einfährt. Der Nutzen liegt also im **Zusammenführen** von Meldungen aus demselben Zug, nicht im Vorwarnen. Eintippen einer 6-stelligen Nummer während einer Kontrolle macht kaum jemand. Scannen (Texterkennung auf dem Gerät, ohne Upload) ist bequemer, aber auch ein Kamerablick im Kontrollmoment.
- **Plattform:** Eintippen überall, Texterkennung auf dem Gerät in iOS/Android gut, im Web mäßig.
- **Rolle:** später als **optionale** Angabe, nur wenn der Feldtest zeigt, dass Nummern innen sichtbar sind und genutzt werden.

**c) WLAN-BSSID im Zug**

- **Befund:** Für die S-Bahn Hamburg fand ich **keinen Beleg für Fahrgast-WLAN in den Zügen** (BR 474/474plus, BR 490). Die gefundenen Quellen zur Modernisierung „474plus“ nennen Wagenübergänge und Fahrgastinformation, kein WLAN. Kostenloses WLAN gibt es laut Handelskammer an den U-Bahn-Haltestellen und in Bussen; für S-Bahn-Bahnsteige fand ich keinen klaren Beleg. ⚠️ Im Zug selbst einmal mit einer WLAN-Scanner-App prüfen.
- **Selbst wenn:** iOS lässt Apps **gar nicht** nach WLANs suchen, nur das gerade verbundene Netz auslesen (mit Sonderberechtigung). Android: höchstens 4 Scans pro 2 min im Vordergrund, mit Standortrecht. Web: unmöglich.
- **Rolle:** nie für Züge. WLAN an Stationen höchstens später als Tunnel-Anker unter Android, niedrige Priorität.

### 2.4 Station-first

**Idee:** Hauptprodukt sind Meldungen **an Stationen**: Bahnsteig, Ausgang, Großkontrolle (Abgangskontrollen sind laut hvv-Beförderungsbedingungen ein normales Kontrollmuster, Recherche 8). Zugmeldungen gibt es zusätzlich, als Band (2.1).

- **Stärke:** Stationskontrollen sind ortsfest, dauern lange (oft 30–90 min) und werden von vielen gesehen. Das gibt **viele unabhängige Meldungen pro Ereignis**, also echte Mehrfachbestätigung. Keine Zuordnung, kein Echtzeit-API. An Knoten (Hbf, Jungfernstieg, Altona) gilt eine Meldung für alle Linien (Recherche 10.5).
- **Schwäche:** Streckenkontrollen im fahrenden Zug erfasst sie nur indirekt.
- **Zu „Zugmeldung nur mit Check-in“:** Eine Pflicht zum Check-in wäre wieder eine Hürde und Zuordnungsquelle. Besser: Eine Zugmeldung braucht nur **Station + Richtung (+ Linie)** zum Zeitpunkt des Tippens. Das ist der Anker für das Band, ohne Fahrtwahl. Ein „Check-in“ ist dann nur ein gespeicherter Anker für Rückfragen.

### 2.5 Zug-Raum

**Idee:** Jede Fahrt ist ein kurzlebiger anonymer Raum. Wer drin ist, meldet dort, Mitreisende korrigieren Fehler.

- **Problem:** Wie kommt man in den richtigen Raum? Über die Wahl der Fahrt (dann ist das alte Problem nur verschoben) oder über BLE (2.3 a, ausgeschlossen).
- **Reparatur durch Mitreisende** braucht ≥ 2 Nutzer pro Zug, also dieselbe hohe Dichte wie BLE.
- **Risiken:** Räume laden zu Chat und Freitext ein (v2 verbietet das), ein Gegner kann den Raum betreten und entwarnen, und eine Anzeige „3 Personen im Raum“ verrät dem Prüfdienst, dass App-Nutzer an Bord sind.
- **Brauchbarer Kern:** ein **impliziter** Raum ohne Oberfläche. Der Server fasst Anker zusammen, die zueinander passen (gleiche Richtung, Zeiten im selben Band, später zusätzlich die Halt-Signatur aus 2.9). Das ist ein Rechenschritt, kein Produkt.

### 2.6 Lean-Start: Telegram-Bot oder Webseite

**Idee:** Vor dem Zugmodell 4–6 Wochen lang mit einfachen Mitteln messen, ob es in Hamburg genug Meldungen gibt.

- **Warum nötig:** Die Hamburger Telegram-Gruppe „Kontrollettis HH“ hat 627 Mitglieder, aber nur 8 Nachrichten in 30 Tagen. Eine Facebook-Gruppe hat über 6.000 Mitglieder. Die Nachfrage ist also **unklar**. Das gesamte Zugmodell lohnt sich nur bei einer Meldedichte, die heute niemand kennt.
- **Umfang:** Web-Seite (PWA) und Telegram-Bot mit Knöpfen (Linie → Station → Bahnsteig / Ausgang / Großkontrolle / im Zug Richtung …), **ohne Freitext**. Anzeige: Stationsliste, Abschnittsanzeige, Band als Text („möglich in S1-Zügen, die ca. 08:42–08:45 in Jungfernstieg ankommen“). Ein Telegram-Kanal postet nur Liniensummen (wie v2 Abschnitt 8).
- **Aufwand:** 1–3 Wochen für eine Person. Kein App-Store, also kein Store-Risiko.
- **Datenschutz:** Telegram sieht Konten und Metadaten. Der Bot speichert Telegram-IDs nur als gesalzenen Hash für das Rate-Limit und löscht Rohmeldungen wie v2. Das muss in der Datenschutzerklärung stehen.
- **Kennzahlen:** Meldungen pro Tag und Linie; Anteil „im Zug“ gegenüber „Station“; Zeit bis zur zweiten unabhängigen Meldung; Anteil bestätigter Meldungen; Band-Trefferquote; wiederkehrende Leser pro Woche; Spitzen bei Großkontrollen.
- **Entscheidungsregeln (Startwerte):**
  - unter 20 Meldungen pro Tag im S-Bahn-Netz nach 6 Wochen: kein Zugmodell, Produkt bleibt Station + Abschnitt.
  - mindestens 30 % „im Zug“ und Median bis zur zweiten Meldung unter 10 min: native App mit Band (Abschnitt 4, Stufe 2).

### 2.7 Vertrauen über GPS-Verweildauer an der Station (Idee des Nutzers)

**Idee:** Wer bei einer gemeldeten Massenkontrolle mehrere Minuten an der Stelle steht, bekommt mehr Vertrauen.

| Frage | Bewertung |
|---|---|
| GPS unterirdisch | Gerade die Hotspots liegen unter der Erde: Hbf (S-Bahn-Gleise im City-Tunnel ⚠️ Gleisnummern prüfen), Jungfernstieg, Altona (S-Bahn-Tunnelstation), Landungsbrücken bis Königstraße, Harburg/Harburg Rathaus, Airport. Dort gibt es **kein GPS**. Das Handy liefert dann den letzten Fix oder eine WLAN-/Funkzellen-Schätzung mit 30–500 m Ungenauigkeit. Bahnsteig, Verteilerebene und Straße darüber sind nicht unterscheidbar. |
| Fälschbarkeit | Android: Schein-Standort über Entwickleroptionen, mit Root auch versteckt. iOS: über Hilfsprogramme oder Zusatzgeräte möglich (iOS 15+ kann Software-Simulation teilweise erkennen). Web: über die Entwicklerwerkzeuge des Browsers trivial. |
| Beweist Stehen eine Schlange? | **Nein.** Stehen bleiben alle Wartenden, Personal, Sicherheitsdienst und die Kontrolleure selbst. Wer eine Kontrolle meidet, geht eher weg. Die Regel belohnt also nicht die ehrlichsten Melder, sondern die, die am längsten dort sind. Ein Gegner vor Ort hat beliebig viel Verweildauer. |
| Datenschutz | Verlangt minutenlanges Standort-Tracking genau am Ort einer Kontrolle. Würde die Spur übertragen, entstünde ein Beleg „Gerät X war während der Kontrolle dort“. |
| Gegenüber unabhängigen Mehrfachmeldungen | Schlechter. Mehrere Meldungen von verschiedenen Konten zu verschiedenen Zeiten sind billiger, funktionieren unterirdisch und brauchen keinen Standort. |

**Brauchbare Variante:** eine **Ortsplausibilität auf dem Gerät**. Die App prüft lokal, ob es in den letzten 15 min einen Standort in der Nähe der gemeldeten Station gab, und sendet nur `plausibel / unbekannt / unplausibel`, nie Koordinaten. `unbekannt` (Tunnel, kein Recht) darf nicht bestrafen. `unplausibel` (Gerät 5 km entfernt) senkt das Gewicht. Das ist ein schwaches Zusatzsignal, kein Vertrauensbonus.

### 2.8 Vertrauen über Live-Selfie (Idee des Nutzers)

**Idee:** In der App spontan ein Selfie mit Menschenmenge oder Kontrolleuren im Hintergrund, eigenes Gesicht automatisch verpixelt. Hier nur Praxis und Technik.

| Frage | Bewertung |
|---|---|
| Wer prüft? | **Menschen:** Moderation rund um die Uhr ist für ein Hobbyprojekt nicht zu leisten. Bis jemand prüft, ist die Meldung (Minuten-Lebensdauer) veraltet. **ML:** Ein Modell erkennt „viele Menschen in einem Bahnhof“, aber nicht „das sind Kontrolleure“. Der Prüfdienst arbeitet oft in Zivil, Uniformen sehen ähnlich aus wie die von Sicherheitsdienst oder Polizei. |
| Wie schnell? | ML auf dem Gerät: unter 1 s. Server-ML: wenige Sekunden plus Upload im Tunnel-Funkloch. Mensch: Minuten bis Stunden. |
| Fälschbarkeit | Foto eines Bildschirms (Moiré-Erkennung hilft nur teilweise), altes Foto über eine manipulierte Kamera-Schnittstelle (gerootete Geräte), beliebige Menschenmenge irgendwo. Ein Gegner fotografiert einfach eine echte Menge. |
| Widerspruch im Entwurf | Das Selfie soll „live und echt“ beweisen, aber das eigene Gesicht wird verpixelt. Damit fällt genau der Teil weg, der Lebendigkeit belegen könnte. |
| Konflikt und Sicherheit | Wer Kontrolleure sichtbar anfilmt, provoziert Ansprache und Streit, oft genau die Person, die gerade kein Ticket hat. Blitz, ausgestreckter Arm und Wegdrehen fallen auf. Das widerspricht der v2-Regel, keine riskanten Handlungen anzustoßen (M-10). |
| Datenschutz für den Melder | Foto + Ort + Zeit ist ein Beleg der eigenen Anwesenheit. Verpixeln versagt bei Profil, Maske, Spiegelung, teilweise sichtbarem Gesicht. Kleidung, Tasche, Tattoos und Mitreisende im Hintergrund bleiben erkennbar. Bei Server-Einbruch oder Herausgabe (etwa in einem Beihilfe-Verfahren) wären das Beweisstücke. Dazu EXIF-Metadaten. |
| Nutzen gegenüber Mehrfachbestätigung | Gering. Kostet 10–20 s im ungünstigsten Moment, also weniger Meldungen. Zwei unabhängige Ein-Tap-Meldungen sind schneller und schwerer zu fälschen als ein Foto. |

**Variante ohne Upload (nur falls überhaupt):** „Kamera-Check“ auf dem Gerät. Die App nimmt ein Bild der Umgebung (nicht das Gesicht), ein lokales Modell prüft „Live-Szene, kein Bildschirm, Innenraum mit Menschen“, das Bild wird **sofort verworfen**, gesendet wird nur `kamera_ok = true` mit Geräte-Attest. Ehrliche Einschätzung: Das hebt die Kosten für Spam vom Sofa aus ein wenig, belegt aber keine Kontrolle und behält das Konflikt-Risiko beim Fotografieren. Empfehlung: **nicht bauen**. Höchstens als Experiment für Großkontroll-Meldungen neuer Konten, wenn Spam nachweislich ein Problem wird.

### 2.9 Weitere Ansätze

**a) Halt-Signatur aus dem Bewegungssensor.** Alle Handys im selben Zug bremsen und fahren in derselben Sekunde an. Das Gerät erkennt Halte mit dem Beschleunigungssensor (geht im Tunnel) und kennt damit die Ist-Zeiten seines Zuges.
- *Allein nützlich:* **Haltezähler im Tunnel.** „Seit Altona 3 Halte“ heißt Landungsbrücken, ohne GPS. Das verbessert den Anker und damit das Band.
- *Mit mehreren Nutzern:* Der Server erkennt Geräte im selben physischen Zug über passende Abfahrtszeiten an 2–3 Halten in Folge. Züge im 2,5-min-Abstand unterscheiden sich klar. Das ist der implizite Zug-Raum aus 2.5, ohne Funkkennung, die andere sehen können.
- *Plattform:* Android mit Vordergrunddienst gut, iOS im Vordergrund gut, im Hintergrund eingeschränkt ⚠️, Web nur bei offener Seite und mit Zustimmung.
- *Datenschutz:* Abfahrtszeiten sind eine Fahrspur. Daher grob runden (5 s), nur während einer aktiven Meldung oder Rückfrage, kurze Löschfrist wie v2 6.4.

**b) Pull statt Push.** „Ich will jetzt ab Altona Richtung Hbf.“ Die App prüft nur die nächsten 2–3 Züge gegen alle Bänder und Stationsmeldungen. Das ist die Nutzerfrage in ihrer einfachsten Form und braucht keine Zugkarte.

**c) Import bestehender Kanäle** (wie FreiFahren und KontrollRadar). Mehr Meldungen beim Start, aber Freitext und unklare Ankerzeiten. Nur mit Kennzeichnung „importiert, ungeprüft“ und niedrigem Gewicht ⚠️ Rechte der Gruppenbetreiber klären.

---

## 3. Bewertung

Skala: hoch / mittel / gering. Bei Aufwand und Nutzerdichte ist „gering“ gut.

| Ansatz | Zuverlässigkeit | Aufwand | Nötige Nutzerdichte | Datenschutz | Nutzen für Fahrgast | Urteil |
|---|---|---|---|---|---|---|
| *Basis: Konzept v2 (Zugstatus je Fahrt)* | mittel, stille Fehler bei Fehlzuordnung | hoch | hoch (Check-ins in vielen Zügen) | mittel (Check-in-Spuren) | hoch, wenn es stimmt | zu früh |
| 2.1 Unsicherheitsband | **hoch**, Fehler fallen breit statt falsch aus, messbar | mittel | gering (1 Meldung reicht) | **gut** (kein Check-in nötig) | hoch auf den nächsten Halten, „1 von 2 Zügen“ im dichten Takt | **Kern** |
| 2.2 Abschnitt/Station-Ampel | hoch, aber grob | gering | gering | gut | mittel (Übersicht, kein Zug) | **ja, als Übersicht** |
| 2.3 a BLE-Erkennung | mittel (Gegenzug am Bahnsteig, iOS-Hintergrund) | hoch | **sehr hoch** (≥ 2 aktive pro Zug) | **schlecht**: Nutzer für jeden Scanner sichtbar | mittel | **nie** |
| 2.3 b Fahrzeugnummer | hoch, wo genutzt | gering bis mittel | hoch (Melder müssen tippen) | gut | gering (Zusammenführen, Zugteil) | später, optional |
| 2.3 c Zug-WLAN | – (kein WLAN gefunden, iOS kann nicht scannen) | mittel | mittel | mittel | – | **nie** |
| 2.4 Station-first | **hoch** (ortsfest, viele Zeugen) | gering | gering | gut | hoch an Knoten, gering für Streckenkontrollen | **Kern** |
| 2.5 Zug-Raum (sichtbar) | mittel | hoch | hoch | mittel bis schlecht (Raumgröße verrät Nutzer) | mittel | nie als Oberfläche, später implizit |
| 2.6 Lean-Start Bot/Web | hoch für Stationen | **gering** | gering | mittel (Telegram-Metadaten) | mittel | **zuerst** |
| 2.7 GPS-Verweildauer | gering (unterirdisch kein GPS, Stehen beweist nichts) | mittel | mittel | schlecht, wenn übertragen | gering | nie als Bonus; nur Plausibilität auf dem Gerät |
| 2.8 Live-Selfie mit Upload | gering (fälschbar, ML erkennt keine Kontrolleure) | hoch | – | **schlecht** | negativ (Zeitverlust, Konflikt) | **nie** |
| 2.8 Kamera-Check ohne Upload | gering bis mittel | mittel | – | gut | gering | wahrscheinlich nie |
| 2.9 a Halt-Signatur | mittel bis hoch | hoch | gering (Haltezähler), mittel (Zusammenführen) | mittel | mittel (bessere Anker im Tunnel) | später |
| 2.9 b Pull-Abfrage | hoch (nutzt Band + Station) | gering | gering | gut | **hoch** | zuerst |

---

## 4. Empfohlene Startvariante: „Stationen + Band“

**Grundsatz:** Die Wahrheit ist der Anker (Ort, Zeit, Richtung). Eine konkrete Fahrt ist nur eine **Beschriftung**, die die App zeigt, wenn das Band schmaler als der halbe Takt ist („wahrscheinlich S1 08:41“). Sonst zeigt sie ehrlich „einer von 2 Zügen“.

### Stufe 1, zuerst (Lean-Start, ca. 4–6 Wochen)
- Web/PWA + Telegram-Bot mit Knöpfen, ganzes S-Bahn-Netz (Stationsmeldungen gelten an Knoten linienübergreifend).
- Meldearten: **Bahnsteig, Ausgang, Großkontrolle** (Station-first) und **im Zug** als Anker „Station + Richtung (+ Linie, Ziel)“, ohne Fahrtwahl und ohne Check-in.
- Anzeige: Stationsliste, Abschnittsanzeige (2.2), Pull-Abfrage „meine nächsten Züge ab hier“ mit Band (2.1, 2.9 b).
- Zustände wie im Review vorgeschlagen: „einzeln gemeldet“, „mehrfach bestätigt“, „widersprüchlich“, „veraltet“. Keine Prozentzahlen, kein Grün.
- Vertrauen nur über **unabhängige Mehrfachmeldungen**, Rate-Limit, Notbremse, Entwarnung schwer (v2 5.4, ohne Gerätestufen).
- Messen nach 2.6 und an den Entscheidungsregeln entscheiden.

### Stufe 2, später (nur wenn die Kennzahlen es tragen)
- Native App (Flutter) mit dem **Band auf dem Linienstreifen** als schraffierte Zone. Züge im Band werden 🟠 „möglich“, 🔴 nur bei frischer Bestätigung im Band.
- Echtzeit (DB Timetables) nur für Abfahrtszeiten und Störungserkennung, **nicht** als Voraussetzung für Warnungen.
- Optionaler „Anker-Check-in“ für Rückfragen („Kontrolleure noch da?“), mit Haltezähler im Tunnel (2.9 a).
- Ortsplausibilität auf dem Gerät als schwaches Signal (2.7), Geräte-Attest, Push als Linien-Abo.

### Stufe 3, noch später (nur mit Messdaten)
- Halt-Signatur zum Zusammenführen von Meldungen aus demselben Zug (impliziter Zug-Raum).
- Fahrzeugnummer per Texterkennung auf dem Gerät, wenn der Feldtest sie innen findet.
- Fahrtgenaue Zugkarte aus v2, wenn die Band-Trefferquote ≥ 80 % liegt und das Band im Median ≤ 1 Zug breit ist.
- Kalibrierung der Parameter (e_meld, e_lauf, Abklingen) aus den Treffern.

### Nie
- BLE-Erkennung der Mitreisenden (macht Nutzer für den Prüfdienst sichtbar, Web unmöglich).
- Foto- oder Selfie-Upload, auch verpixelt.
- GPS-Verweildauer als Vertrauenswährung oder Übertragung von Standortspuren.
- WLAN-Scans in Zügen.
- Sichtbare Zug-Räume mit Teilnehmerzahl oder Chat; Teamverfolgung (wie im Review).

### Warum robuster als Konzept v2

1. **Kein Einzelpunkt Zugzuordnung.** In v2 erzeugt eine falsche Fahrt einen stillen Fehler (richtiger Zug zeigt ⚪). Das Band warnt im Zweifel zwei Züge statt den falschen.
2. **Verspätungsfest ohne Echtzeit.** Die Ankerzeit enthält die Verspätung schon. Am Bahnsteig reicht „jetzt einfahrender Zug“. Echtzeit wird ein Plus, keine Voraussetzung.
3. **Funktioniert mit wenigen Nutzern.** Eine einzige Meldung reicht für Band und Station. v2 braucht Check-ins in vielen Zügen für Datenlage, Rückfragen und Crowd-Verspätung.
4. **Setzt auf die Meldungen, die es oft gibt.** Stationskontrollen sind ortsfest und haben viele Zeugen, also echte Mehrfachbestätigung.
5. **Prüfbar.** Die Band-Trefferquote zeigt, wie gut die App ist, und stellt die Parameter ein. Für „richtige Fahrt“ gibt es in v2 keine einfache Messung.
6. **Weniger Daten, weniger Angriffsfläche.** Kein Pflicht-Check-in, keine Spuren, kein Funk, keine Fotos.
7. **Billiger Irrtum.** Der Lean-Start kostet Wochen statt Monate und zeigt, ob Hamburg überhaupt genug Meldungen liefert, bevor das teure Zugmodell gebaut wird.

**Was dabei verloren geht:** Das Alleinstellungsmerkmal „jeder Zug einzeln“ kommt erst in Stufe 3 und nur dort, wo die Daten es tragen. Im City-Tunnel zur HVZ wird die App oft „einer von 2 Zügen“ sagen. Das ist weniger beeindruckend, aber wahr.

---

## Quellen zur WLAN-Prüfung

- [Rail Journal: Refurbished Hamburg S-Bahn train revealed](https://www.railjournal.com/passenger/commuter-rail/refurbished-hamburg-s-bahn-train-revealed/) (474plus: Wagenübergänge, Fahrgastinformation, kein WLAN genannt)
- [Rail Journal: Hamburg to exercise option for 64 S-Bahn trains](https://www.railjournal.com/fleet/hamburg-to-exercise-e400m-option-for-64-s-bahn-trains/)
- [Handelskammer Hamburg: Öffentliches WLAN](https://www.handelskammer-hamburg.de/innovation-neue-maerkte/digitalisierung/oeffentliches-wlan-6842500) (WLAN an U-Bahn-Haltestellen und in Bussen)
- [ifun.de: Freies WLAN in Hamburger Bussen und U-Bahn-Stationen](https://www.ifun.de/freies-wlan-in-hamburger-bussen-und-u-bahn-stationen-92011/)
- Websites der S-Bahn Hamburg und Wikipedia waren aus dieser Umgebung nicht erreichbar. ⚠️ Deshalb vor Ort prüfen.
