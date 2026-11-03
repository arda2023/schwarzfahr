# Lösungen S: Veraltete und falsche Meldungen

Bezug: [`01_veraltete_meldungen.md`](01_veraltete_meldungen.md), [`../KONZEPT.md`](../KONZEPT.md), [`../recherche/sbahn_hamburg_recherche.md`](../recherche/sbahn_hamburg_recherche.md).
Die Bausteine **Q1–Q6** sind am Ende unter „Querschnitt“ genau beschrieben. Die Szenarien verweisen nur darauf.
Fahrzeiten sind nach Recherche Abschnitt 3 korrigiert, wo die Szenarien abweichen.

Grundsatz für alle Fälle: Die App weiß nur, was jemand gesehen hat. Sie kann Beobachtungen gut fortschreiben, aber fehlende Beobachtungen nicht ersetzen. Jede Anzeige sagt deshalb, **wie alt** die letzte Sichtung ist und **ob überhaupt jemand an Bord** ist, der melden könnte.

---

### S-01 Melder steigt aus, Kontrolleur verschwindet unbemerkt
*Korrektur:* Fahrzeiten ab Wedel 08:12: Rissen 08:16, Blankenese 08:25, Hochkamp 08:28, Klein Flottbek 08:30. 08:12 ist HVZ. Ab Blankenese fährt die S1 dann etwa alle 5 Minuten (Verstärker ab Blankenese), B wartet also eher 5 als 10 Minuten.

- **Lösung:**
  1. **Abklingen pro Halt statt Timer (Q1).** Die Meldung verliert an jedem Halt an Gewicht, weil Kontrolleure an jedem Halt aussteigen können. Mit den Startwerten (p₀ = 0,85, s = 0,85 pro Halt, 0,7 an Knoten) ergibt sich ohne weitere Information: Rissen 0,85 → Blankenese 0,52 → Hochkamp 0,44. B sieht in Klein Flottbek **🟠 „möglich“**, nicht 🔴.
  2. **Ausstiegsdialog für A (Q5).** Erkennt die App den Ausstieg (6.5) oder drückt A „Ich bin ausgestiegen“, fragt sie mit einem Tap: „Kontrolleure noch im Zug? [noch drin] [auch raus] [weiß nicht]“. „Noch drin“ ist eine Sichtung in Blankenese (p → 0,90). Nach Hochkamp sind es 0,77. Weil niemand mehr an Bord ist, bleibt es trotzdem bei **🟠 „wahrscheinlich noch an Bord“**. 🔴 gibt es nur bei frischer Sichtung (Q1).
  3. **Anzeige auf Bs Abfahrtstafel (Q2):** „🟠 wahrscheinlich · zuletzt gesehen Blankenese, vor 1 Halt · seitdem niemand an Bord, der melden kann“. Direkt darunter steht der nächste Zug mit seinem Status und seiner Datenlage („08:35 · in 5 min · ⚪ keine Meldung · Nutzer an Bord“). B sieht beide Optionen nebeneinander.
  4. **B als neuer Beobachter.** Steigt B trotzdem ein und checkt ein, kommt sofort die Rückfrage „Kontrolleure in deinem Zugteil?“ (Q5). Ein „Nein“ senkt p für Bs Zugteil, und die Nutzer ab Othmarschen sehen schon besseren Stand.
  5. **Ausbreitung (Q4).** Was am Halt Hochkamp „aus dem Zug fließt“, landet als schwacher Hinweis bei Station Hochkamp und beim nächsten Gegenzug ab Hochkamp. Dort war F1 tatsächlich.
  6. **Kein Dauerfehler über die ganze Linie.** Ohne neue Meldung fällt p weiter: Klein Flottbek 0,65, Othmarschen 0,56, Bahrenfeld 0,47, Ottensen 0,40, Altona (Knoten) 0,28, Königstraße 0,24 → ⚪. Das „wiederholt sich an jeder Station“ aus dem Szenario wird mit jedem Halt schwächer und endet spätestens an der Endstation.
- **Antwort auf die Leitfrage („Was dann? Eigenes Risiko?“):** Ja. Ob B einsteigt, ist Bs eigene Entscheidung und Bs eigenes Risiko, in beide Richtungen: Wartet B, verliert B 5 bis 10 Minuten umsonst. Steigt B ein und F1 wäre doch noch drin, zahlt B 60 €. Dass F1 in Hochkamp ausgestiegen ist, kann die App nicht wissen, wenn es niemand sieht. Sie kann aber verhindern, dass B einer **veralteten Gewissheit** folgt. Statt eines roten Punkts sieht B eine ehrliche Schätzung mit Alter („vor 1 Halt“) und dem Hinweis, dass niemand mehr an Bord melden kann. Die App entscheidet nie für B und verspricht nichts. Im Detailblatt steht: „Die App kann sich irren. Sicher ist nur ein gültiges Ticket.“ Das passt auch zur rechtlich gebotenen neutralen Tonlage (KONZEPT 8).
- **Status:** 🟡 teilweise
- **Restrisiko:** Unbemerktes Aussteigen ohne Beobachter an Bord ist grundsätzlich nicht erkennbar. Die App macht das sichtbar: „zuletzt gesehen vor N Halten“, „niemand an Bord“ und 🟠 statt 🔴. Die Abklingwerte sind anfangs geschätzt und werden kalibriert (Q1).

### S-02 Falsch-negativ: Kontrolleure steigen still zu
*Korrektur:* Die S1 fährt nicht über Hammerbrook. Ab Altona 17:40 ist sie um 17:53 am Hbf und danach am Berliner Tor.

- **Lösung:**
  1. **⚪ aufteilen (Q2):** „⚪ keine Meldung · Nutzer an Bord“ und „◌ keine Daten · niemand an Bord“. Damit liest niemand „keine Meldung“ mehr als „frei“, wenn gar niemand hinschaut.
  2. **„Ich werde gerade kontrolliert“** als eigener, besonders schneller Meldetyp (`KONTROLLIERT_GERADE`, hohes Gewicht, Q5). Kontrollierte Fahrgäste mit Ticket sind die häufigste und verlässlichste Quelle. Sie sollen ohne Nachdenken melden können, auch nachträglich innerhalb von 2 Halten.
  3. **Inferenz aus Stationsmeldungen (Q4):** Lag in Altona kurz vorher eine Bahnsteigmeldung vor, bekommen die dort abfahrenden Züge den Hinweis „möglich (Umstieg)“.
  4. **Grundrisiko statt Leere:** Aus den aggregierten Statistiken (KONZEPT 7: nur Zahlen pro Station und Wochentag) kann die Abfahrtstafel einen dezenten Satz zeigen, z. B. „Abschnitt Hbf–Barmbek: werktags 17–18 Uhr oft gemeldet“. Das ist keine Meldung und hat keine Farbe.
  5. **Onboarding und Detailblatt:** einmalig erklären, dass ⚪ „wir wissen nichts“ bedeutet.
- **Status:** ❌ nicht lösbar (im Kern), gemildert
- **Restrisiko:** Ohne Beobachter gibt es keine Information. Das ist der häufigste Fall und bleibt es. Die App kann nur offen sagen, wie dünn die Datenlage ist (◌, Grundrisiko-Satz), und darf ⚪ nie als Entwarnung formulieren.

### S-03 Flügelung Ohlsdorf: Meldung landet im falschen Zugteil
- **Lösung:**
  1. **Zugteil aus dem Check-in ableiten (Q3).** Wer vor Ohlsdorf in einen Flügelzug eincheckt, wählt ohnehin sein Ziel. Airport heißt vorderer Teil, Poppenbüttel heißt hinterer Teil. So ist der Zugteil des Melders fast immer ohne Extra-Tap bekannt.
  2. **Meldeblatt auf Flügelzügen vor Ohlsdorf:** „In meinem Zugteil / im anderen Zugteil / weiß nicht“. Das deckt den Fall ab, dass A Kontrolleure am Bahnsteig in den anderen Teil einsteigen sieht.
  3. **Eigenes p je Teil.** Bis Ohlsdorf zeigt die Karte den Zug mit dem höheren Wert und dem Hinweis „vorderer Teil“. Ab Ohlsdorf geht jedes p an seine Fahrt (vorne → Airport-fahrtKey, hinten → Poppenbüttel-Fahrt). `partnerOf` bleibt nur die Klammer für den physischen Zug.
  4. **Teil unbekannt:** Beide Teile bekommen p × 0,5 und den Text „🟠 in einem der beiden Teile“. Eingecheckte Nutzer in beiden Teilen bekommen nach Ohlsdorf die Rückfrage (Q5). Die erste Antwort klärt meist beide Teile.
  5. **Ohlsdorf ist ein Knoten (s = 0,7, Q1):** Beim Teilen können Kontrolleure auf dem Bahnsteig in den anderen Teil wechseln. Ein kleiner Anteil fließt deshalb in den Partnerteil (Q4, „möglich“).
  6. HVZ-Verstärker Blankenese ⇄ Poppenbüttel werden nicht geflügelt. Das Datenskript erkennt sie (6.1) und bietet dort keine Teilwahl an.
- **Status:** ✅ gelöst, sofern Zugteil oder Ziel des Melders bekannt ist. Sonst 🟠 mit klarer Kennzeichnung.
- **Restrisiko:** Ein Wechsel des Zugteils während des Halts in Ohlsdorf bleibt unsichtbar. Er wird als „möglich“ im Partnerteil angezeigt, nicht als 🔴.

### S-04 Umstieg der Kontrolleure am Knoten
*Korrektur:* Die S3 braucht von Harburg bis Hbf 16 Minuten (Abfahrt 10:10, Ankunft ≈ 10:26). S1 und S3 fahren ab Hbf beide durch den City-Tunnel nach Altona. Der Wechsel in die S1 Richtung Wedel ist plausibel, ebenso die S1 Richtung Ohlsdorf.

- **Lösung:**
  1. **Knoten-Abklingen (Q1):** Am Hbf ist s = 0,7. Die S3 fällt nach dem Hbf ohne neue Sichtung auf 🟠 und nach einigen Halten auf ⚪. Sie bleibt nicht bis Pinneberg rot.
  2. **Ausbreitung (Q4):** Der Anteil, der den Zug am Hbf „verlässt“, geht als Bahnsteighinweis an den Hbf und als „möglich (Umstieg)“ an alle Züge, die dort in den nächsten 10 Minuten abfahren. Dazu gehören S1 beide Richtungen, S3 Gegenrichtung, S5, S2 und S7. Das ist bewusst nur ein gestrichelter Hinweis, nie 🔴.
  3. **Rückfrage (Q5)** an eingecheckte Nutzer in der S3 nach dem Hbf: „Kontrolleure noch hier?“. Ein „Nein“ senkt p der S3 schnell und verstärkt den Umstiegshinweis.
  4. **Wording gegen Ausweichen:** Die App empfiehlt nie einen „besseren“ Zug und kennt keine Funktion „Alternative ohne Meldung“. Auf Zügen mit Umstiegshinweis steht: „Kontrolleure könnten am Hbf umgestiegen sein“.
- **Status:** 🟡 teilweise
- **Restrisiko:** Welchen Zug F1 und F2 nehmen, ist ohne Beobachter nicht bestimmbar. Die App zeigt nur die Möglichkeit. Die Ausbreitung darf das Netz nicht flächig orange färben (Begrenzung siehe Q4). Sie wird nur auf dem Gerät gerechnet und nicht gespeichert, damit keine Bewegungsprofile entstehen (KONZEPT 8).

### S-05 Verfall nach Zeit statt nach Ereignis
*Korrektur:* Die S3 fährt Pinneberg ⇄ Neugraben und nur in Wochenendnächten bis Stade. Um 07:30 fährt sie als Langzug (werktags 5–19 Uhr). Im KONZEPT steht keine 20-Minuten-Regel für Zugmeldungen, sondern „15 min → 🟠, 30 min weiter → ⚪“ (5.2). Das Problem besteht dort genauso: Bei Abfahrt 07:30 in Pinneberg ist der Zug um 08:15 bei Wilhelmsburg schon ⚪.

- **Lösung:**
  1. **Timer aus 5.2 streichen**, Abklingen pro Halt (Q1). Lange Linien sind damit kein Sonderfall mehr: Es zählt, wie viele Gelegenheiten zum Aussteigen es gab, nicht wie viele Minuten vergangen sind.
  2. **Bestätigungen setzen die Kette zurück.** Jede Bestätigung oder Rückfrage-Antwort „Ja“ (Q5) ist eine neue Sichtung am jeweiligen Halt. Kontrolliert F1 bis Neugraben und ist ab und zu jemand an Bord, bleibt der Zug 🔴 bis zum Ende.
  3. **s je Linie und Zuglänge:** Bei Langzügen (S3, 9 Wagen) brauchen Kontrolleure länger für den Zug und bleiben im Schnitt länger an Bord. Startwert dort s = 0,9. Danach kalibrieren (Q1).
  4. Eine Zeitkomponente bleibt nur für **stehende** Situationen: Bahnsteigmeldungen (S-06) und Züge, die laut Crowd-Verspätung lange an einem Halt stehen (dann zählt die Standzeit als zusätzlicher halber Halt je 5 Minuten).
- **Status:** ✅ gelöst (das strukturelle Problem der festen Ablaufzeit)
- **Restrisiko:** Bleibt F1 lange an Bord und bestätigt niemand, fällt die Meldung trotzdem nach etwa 6 bis 9 Halten unter die Schwelle. Das ist die Datenlücke aus S-10. Sichtbar wird sie durch „zuletzt gesehen vor N Halten“.

### S-06 Stationsmeldung bleibt hängen
*Korrektur:* Am Jungfernstieg halten die S-Bahnen S1, S3 und S7, nicht die S2. Betroffen sind also S1, S3 und S7. Dazu kommt die U-Bahn, die nicht Teil der App ist.

- **Lösung:**
  1. **Kürzere, zweistufige Lebensdauer:** 🔴 nur bis 10 Minuten nach der letzten Sichtung, danach 🟠 mit Halbwertszeit 10 Minuten, unter 0,25 dann ⚪. Das ersetzt „verblassen nach 20, weg nach 45 min“ (5.2). Startwerte, später kalibrieren.
  2. **Rückfragen an Durchreisende (Q5):** Wer eingecheckt am Jungfernstieg aussteigt, bekommt „Noch Kontrolleure am Bahnsteig?“. Wer dort nur durchfährt, bekommt beim Halt „Siehst du Kontrolleure am Bahnsteig?“ (schwächeres Gewicht, weil der Blick aus dem Fenster begrenzt ist). Ein „Nein“ von 2 Personen senkt die Meldung deutlich. Bei vielen Durchfahrten im City-Tunnel (bis 24 Züge pro Stunde und Richtung) kommen schnell Antworten zusammen.
  3. **Ausbreitung (Q4):** Eine abklingende Bahnsteigmeldung gibt einen schwachen Hinweis „möglich“ an die Nachbarstationen (Stadthausbrücke, Hbf). Ziehen Kontrolleure zu Fuß oder mit dem nächsten Zug weiter, ist das wahrscheinlich.
  4. **Optionaler Ort ohne Freitext:** „Bahnsteig“ oder „Ausgang/Zwischenebene“. Bei großen Stationen hilft das dem Nutzer, ohne Personen zu beschreiben.
- **Status:** 🟡 teilweise
- **Restrisiko:** Wenn niemand vorbeikommt, der antwortet, bleibt eine Bahnsteigmeldung bis zu etwa 20 Minuten als 🟠 sichtbar. Das Alter steht immer dabei („vor 18 min“). Dass F1 bis F3 an der Stadthausbrücke stehen, sieht die App nur, wenn es dort jemand meldet.

### S-07 Verspätung verschiebt die berechnete Zugposition
*Korrektur:* Mit 8 Minuten Verspätung ist die S1 ab Wedel 08:12 statt um 08:33 um 08:41 in Othmarschen. Eine Zuordnung nach Fahrplan landet beim **Folgezug** (08:22 ab Wedel oder einem Verstärker ab Blankenese), nicht beim Vorderzug. Das KONZEPT verlangt für Zugmeldungen einen Check-in (5.3 Nr. 2). Der Fehler entsteht also schon beim Check-in, etwa wenn die Abfahrtstafel in Wedel die verspätete 08:12 schon als „abgefahren“ führt und A die 08:22 antippt.

- **Lösung:**
  1. **Abfahrtstafel nach Reihenfolge, nicht nach Uhr (Q6).** Auf einem Gleis überholen sich S-Bahnen im Regelbetrieb nicht. Ein Zug verschwindet erst von der Tafel, wenn ein Crowd-Signal ihn als abgefahren meldet oder die geschätzte Verspätung abgelaufen ist. Bis dahin steht dort „08:12 · noch nicht als abgefahren gemeldet“.
  2. **Check-in prüfen und korrigieren:** Nach dem Losfahren gleicht das Handy die GPS-Spur mit den Kandidaten ab (6.3 Nr. 2), einschließlich Crowd-Verspätung. Passt die 08:12 mit +8 besser, fragt die App: „Bist du in der 08:12 (+8 min)?“. Bei Bestätigung sendet sie `KORREKTUR` und bindet alle eigenen Meldungen dieser Fahrt neu (das Gerät kennt die IDs seiner Meldungen).
  3. **Jede Meldung ist auch ein Positionssignal:** „Fahrt X, Othmarschen, 08:41“ liefert die Verspätung mit (Feld `verspaetung_sek`). Mit dem Überholverbot erben die Folgezüge mindestens die Verspätung, die der Takt erzwingt.
  4. **Mehrdeutige Fälle:** Kommen zwei Fahrten innerhalb von ±3 Minuten in Frage und ist keine bestätigt, speichert die App die Meldung mit `kandidaten` (z. B. 0,6/0,4). Beide Züge zeigen 🟠 „in einem dieser Züge“, bis ein eingecheckter Nutzer es klärt.
  5. Phase 3 (DB Timetables) senkt den Fehler weiter, ist aber keine Voraussetzung.
- **Status:** 🟡 teilweise (✅, sobald Crowd-Verspätung oder DB-Daten für den Zug vorliegen)
- **Restrisiko:** Im City-Tunnel und im Harburger Tunnel gibt es kein GPS. Ohne andere Nutzer bleibt dort eine Unschärfe von einem Zug. Sichtbar wird sie durch den Kandidaten-Split (🟠 „einer dieser Züge“) statt einer falschen 🔴-Zuordnung.

### S-08 Kontrolleure fahren gegenläufig zurück
- **Lösung:**
  1. **Neugraben als Knoten mit niedrigem s (Q1):** Hier enden die S3 und Teile der S5, und Kontrolleure wenden hier wahrscheinlich oft. Startwert 0,6. Kalibriert wird über die aggregierte Statistik „AUSGESTIEGEN_IN je Station“ (das sind die Wendestationen).
  2. **Ausbreitung in die Gegenrichtung (Q4):** Was in Neugraben „aussteigt“, geht als Bahnsteighinweis an Neugraben und als „möglich (zurückgefahren?)“ an alle stadtwärts abfahrenden S3 und S5 der nächsten 15 Minuten. Das ist genau der Abschnitt der Morgenpendler.
  3. **Die S5 bleibt ein Zug.** Die im GTFS geteilte S5-Fahrt (Elbgaustraße–Neugraben und Neugraben–Stade) wird im Datenskript zusammengefügt (6.1 Nr. 5). Neugraben ist dann nur ein Halt mit niedrigem s, kein künstliches Fahrtende.
  4. **Rückfrage (Q5)** an eingecheckte Nutzer im Zug Richtung Stade nach Neugraben und im stadtwärtigen Zug ab Neugraben.
- **Status:** 🟡 teilweise
- **Restrisiko:** Ohne Beobachter im Gegenzug bleibt es beim Hinweis „möglich“. Die App weiß nicht, ob F1 die S3 oder die S5 zurück nimmt oder in Neugraben bleibt.

### S-09 Langer Zug: Ein Melder sieht nur einen Wagen
*Korrektur:* Langzüge mit 3 Einheiten fahren planmäßig nur auf der S3 (werktags etwa 5–19 Uhr). Die S1 fährt höchstens als Vollzug mit 2 Einheiten. Das Szenario gilt aber genauso für einen S1-Vollzug (F1 vorne, F2 hinten, Landwehr/Hasselbrook) und erst recht für die S3. Wichtig: Zwischen den Einheiten gibt es während der Fahrt keinen Durchgang, auch nicht bei den begehbaren 474plus und 490. Kontrolleure wechseln die Einheit nur am Halt.

- **Lösung:**
  1. **p pro Zugteil auch außerhalb der Flügelung (Q3):** Optional „vorne / hinten“ (bei der S3 tagsüber auch „Mitte“). Die Angabe ist ein Tap und vorausgewählt, wenn der Nutzer sie bei dieser Fahrt schon einmal gemacht hat.
  2. **Entwarnungen gelten nur für den eigenen Teil.** `AUSGESTIEGEN_IN` und `NICHT_MEHR_DA` senken p stark (LR 0,15) nur für den Zugteil des Melders. Ohne Teilangabe wirkt eine Entwarnung auf den ganzen Zug nur halb so stark (LR 0,4). Der Zug fällt also nicht auf ⚪, sondern auf 🟠.
  3. **Abfrage beim Melden einer Entwarnung:** „Alle ausgestiegen? [ja, alle, die ich gesehen habe] [weiß nicht]“. Eine Personenzahl oder Beschreibung wird nicht abgefragt.
  4. **Anzeige:** „Hinten: ausgestiegen in Landwehr · vorne: unbekannt, zuletzt Meldung vor 3 Halten“. Damit wirkt eine Entwarnung nicht mehr wie ein „alles frei“.
  5. Kontrolleure arbeiten meist im Team (Recherche 8). Eine Entwarnung für 1 Person in einem Teil sagt wenig über den Rest.
- **Status:** 🟡 teilweise
- **Restrisiko:** Die Zuglänge (Kurz-, Voll-, Langzug) steht nicht im GTFS. Wenn niemand im anderen Teil sitzt, bleibt dieser unbeobachtet. Die App zeigt das als 🟠 „unbekannt“.

### S-10 Lücke im Abend- oder Nachtverkehr
*Korrektur:* Ab Eidelstedt ist die S3 in etwa 13 Minuten in Pinneberg. Die Meldung endet also um ≈ 23:56 mit der Fahrt und hängt nicht 40 Minuten am Zug. Die 40 Minuten Funkstille betreffen die Linie insgesamt. Um diese Zeit fährt die S3 im 20-Minuten-Takt.

- **Lösung:**
  1. **Datenlage als zweite Achse (Q2):** Eingecheckte Geräte senden ein anonymes `ANWESEND`-Signal pro Fahrt (rotierendes Token, keine Geräte-ID, Löschung bei Fahrtende). Züge ohne Beobachter zeigen ◌ „keine Daten“ statt ⚪. Die Linie bekommt ein Banner: „S3 gerade: kaum Nutzer, Anzeige wenig aussagekräftig“.
  2. **Rote Meldung ohne Beobachter:** Nach dem Ausstieg von A ist der Zug höchstens 🟠 mit „zuletzt gesehen Stellingen 23:41 · seitdem niemand an Bord“. Das Abklingen pro Halt läuft weiter (Q1).
  3. **Fahrtende Pinneberg (Q4):** Die Restwahrscheinlichkeit geht an die Station Pinneberg und an die Rückfahrt ab Pinneberg (Wende).
  4. Abends sinkt s nicht automatisch. Getrennte Kalibrierung nach Tageszeit (Q1).
- **Status:** 🟡 teilweise (nur Transparenz)
- **Restrisiko:** Bei dünner Datenlage weiß die App wenig. Das lässt sich nicht beheben, nur ehrlich anzeigen. Zum Datenschutz: Die Anwesenheit wird nur grob angezeigt („Nutzer an Bord“ / „niemand“), nie als genaue Zahl.

### S-11 Zivilkontrolleure sind noch nicht ausgewiesen
- **Lösung:**
  1. **Anwesenheit ist nie Entlastung (Q2):** Wording „⚪ keine Meldung · Nutzer an Bord“, im Detailblatt „Kontrollen in Zivil sind erst beim Ausweisen erkennbar“. Kein Grün, kein „frei“, kein Häkchen.
  2. **Schweigen zählt fast nichts:** Passive Anwesenheit ohne Meldung hat LR 0,95 pro Halt (Q1). Sie senkt einen bestehenden Verdacht kaum und erzeugt von sich aus nie Entwarnung.
  3. **`KONTROLLIERT_GERADE` als schnellster Meldetyp (Q5):** Sobald F1 und F2 sich vor Hasselbrook ausweisen, ist eine Meldung mit einem Tap möglich. Die Nutzer ab Hasselbrook und Wandsbeker Chaussee haben die Warnung damit 1 bis 2 Minuten vor Ankunft.
- **Status:** ❌ nicht lösbar (im Kern), gemildert
- **Restrisiko:** Bis zum Ausweisen gibt es nichts zu sehen. Die Fahrgäste bis Berliner Tor bleiben ungewarnt. Die App zeigt nur ehrlich, dass sie Zivilkontrollen nicht vorher erkennen kann.

### S-12 Falsche oder missverstandene Meldung
- **Lösung:**
  1. **Meldetypen unterscheiden:** „Kontrolleure gesehen“ (p₀ 0,85 mit Check-in) und „Ich werde gerade kontrolliert“ (p₀ 0,95). Das Meldeblatt weist kurz darauf hin: „Melde, wenn nach Fahrkarten gefragt wird oder ein Prüfgerät zu sehen ist.“ Ein allgemeiner Hinweis, keine Personenbeschreibung.
  2. **Widerspruch mit Grund (Q5):** Bei der Rückfrage gibt es neben „Ja / Nein“ die Antwort „Nein, nur Sicherheit/Polizei“ (`FEHLALARM`, LR 0,2). Eine solche Antwort von einem Eingecheckten senkt 🔴 sofort auf 🟠, zwei Antworten auf ⚪.
  3. **Vertrauen des Geräts (5.3 Nr. 4):** Wird eine Meldung als Fehlalarm widersprochen und nie bestätigt, sinkt das Gewicht künftiger Meldungen dieses Geräts leicht. Ein einzelner Irrtum führt nicht zur Sperre.
  4. Ohne Widerspruch klingt die Meldung pro Halt ab (Q1) und ist nach etwa 6 bis 9 Halten weg.
- **Status:** ✅ gelöst (im Rahmen des Erkennbaren)
- **Restrisiko:** Sitzt kein zweiter Nutzer in der Nähe, wirkt ein Irrtum einige Halte lang. Die Schwere ist niedrig (Zeitverlust), und der Zug zeigt nach dem Ausstieg des Melders ohnehin nur 🟠.

### S-13 Zugtausch oder Ausfall: Meldung wandert ins Leere
- **Lösung:**
  1. **Neuer Ereignistyp `ZUG_ENDET_HIER` (Q5):** Ein Button in der Fahrt-Leiste (4.5): „Zug endet hier / fällt aus“. Er wird zusätzlich automatisch erkannt: Steigen mehrere eingecheckte Geräte (z. B. ≥ 2 oder ≥ 50 % der Anwesenden) an derselben Nicht-Endstation aus und checken innerhalb von 10 Minuten in den Folgezug ein, gilt die Fahrt ab dort als „endet vorzeitig (vermutet)“.
  2. **Übertragung statt Phantom:** Bei vorzeitigem Ende ist s = 0. Die Wahrscheinlichkeit geht zu 70 % auf den Folgezug derselben Richtung (🟠 „mit umgestiegen?“) und zu 30 % auf die Station Harburg (Q4). Der Phantomzug wird ausgegraut: „Fahrt endet laut Nutzern in Harburg“.
  3. **Rückfrage an den Melder,** wenn er selbst in den Folgezug eincheckt: „Sind die Kontrolleure mit umgestiegen? [ja] [nein] [weiß nicht]“.
  4. Phase 3: Ausfälle aus DB Timetables (`/fchg`) setzen das Fahrtende verlässlich.
- **Status:** 🟡 teilweise
- **Restrisiko:** Bei einer Störung mit wenigen Nutzern bleibt das Ende der Fahrt unerkannt, bis DB-Echtzeitdaten angebunden sind. Ob F1 mit umsteigt, bleibt geschätzt (🟠).

### S-14 Meldung zur falschen Linie am Gemeinschaftsabschnitt
*Korrektur:* An der Sternschanze hält die S3 nicht (dort halten S2, S5, S7). In Diebsteich halten S3 und S5, und die S2 endet dort. Die S7 hält in Diebsteich nicht. Realistisch ist also die Verwechslung S3/S5 in Diebsteich oder S5/S7/S2 an der Sternschanze.

- **Lösung:**
  1. **Zug wählen statt Linie tippen (Q6):** Das Meldeblatt „Am Bahnsteig → Kontrolleure stiegen in einen Zug“ zeigt nur die Züge, die gerade da sind oder in den letzten 3 Minuten abgefahren sind, mit **Linie + Ziel + Zeit** („S5 → Neugraben · gerade abgefahren“, „S3 → Pinneberg · vor 2 min“). Das Ziel steht am Zugzielanzeiger und wird in Eile eher richtig gelesen als die Liniennummer.
  2. **Automatische Stationsmeldung dazu:** Wer Kontrolleure am Bahnsteig einsteigen sah, erzeugt immer auch eine kurze Bahnsteigmeldung. Bei einer Linienverwechslung warnt so wenigstens die Station.
  3. **„Weiß nicht welcher“:** speichert `kandidaten` für alle Züge der letzten 3 Minuten (p aufgeteilt, alle 🟠 „in einem dieser Züge“).
  4. **Korrektur innerhalb von 3 Minuten** über „Falscher Zug? Ändern“ (`KORREKTUR`). Danach klären Rückfragen an Eingecheckte in beiden Zügen (Q5) den Fall: Ein „Nein“ im S3 und ein „Ja“ im S5 verschieben die Meldung.
- **Status:** 🟡 teilweise
- **Restrisiko:** Wählt der Melder sicher, aber falsch, und sitzt in keinem der beiden Züge ein Nutzer, bleibt der Fehler bis zum Abklingen bestehen. Die mitgelieferte Bahnsteigmeldung begrenzt den Schaden.

---

## Querschnitt – die wichtigsten Änderungen am Konzept

### Q1 Präsenzmodell: Wahrscheinlichkeit pro Zugteil, Abklingen pro Halt
**Ersetzt** die Timer in KONZEPT 5.2 (15/30 min) und die Formel in 5.3 Nr. 5. Konkretisiert 11.4. **Löst** S-01, S-04, S-05, S-08, S-10, S-12 (teilweise S-03, S-06).

- Zustand je physischem Zug und Zugteil: `p` (Wahrscheinlichkeit, dass Kontrolleure an Bord sind), `letzteSichtung` (Station und Halt-Index) und `beobachterAnBord` (ja/nein, aus `ANWESEND`).
- Start bei `IM_ZUG`: p₀ = 0,85 mit Check-in und 0,6 ohne Check-in oder aus dem Telegram-Bot, jeweils mal Gerätevertrauen.
- **An jedem Halt:** `p ← p · s(halt)`. Startwerte: s = 0,85 an normalen Halten, 0,7 an Knoten (Altona, Diebsteich, Dammtor, Hbf, Berliner Tor, Jungfernstieg, Ohlsdorf, Harburg, Bergedorf), 0,6 in Neugraben und 0 an der Endstation bzw. bei vorzeitigem Ende (dann Q4). Bei S3-Langzügen ist s um 0,05 höher.
- **Beobachtungen nach Bayes in Log-Odds:** `logit(p) ← logit(p) + ln(LR)`. Start-LR: `BESTÄTIGT` 6 mit Check-in, 3 ohne · `KONTROLLIERT_GERADE` 10 · `KEINE_GESEHEN` (eigener Teil) 0,6 · `NICHT_MEHR_DA` 0,4 · `AUSGESTIEGEN_IN` 0,15 für den eigenen Teil, 0,4 ohne Teilangabe · `FEHLALARM` 0,2 · passive Anwesenheit pro Halt 0,95.
- **Anzeige:** 🔴 nur bei p ≥ 0,6 **und** frischer Sichtung (Beobachter an Bord oder Sichtung am vorigen Halt). Sonst 🟠, mit dem Wort „wahrscheinlich“ ab 0,6 und „möglich“ ab 0,25. Darunter ⚪/◌. Immer mit „zuletzt gesehen: X, vor N Halten“. Prozentzahlen höchstens im Detailblatt, weil sie Genauigkeit vortäuschen.
- **Gleiches Ergebnis auf jedem Handy:** Die Parameter kommen versioniert in `sbahn-hh.json` (6.1). Jedes Gerät rechnet aus denselben Ereignissen dasselbe p. Das Backend bleibt so dumm wie in 7 geplant.
- **Kalibrierung:** Aggregiertes Histogramm „Halte zwischen Meldung und `AUSGESTIEGEN_IN`“ je Linie und Tageszeit, zensiert bei Fahrtende ohne Information. Eine Kaplan-Meier-Schätzung daraus ergibt s je Station. Nur Aggregate werden behalten, das passt zum Löschjob nach 24 h (7).

### Q2 Zwei Achsen: Schätzung und Datenlage
**Ändert** 4.1, 4.2, 4.3 und die ⚪-Regel in 5.2. **Löst** S-02, S-10, S-11 (hilft bei S-01).

- Neuer Zustand ◌ „keine Daten“: Niemand ist an Bord und es gab in den letzten 30 Minuten keine Aktivität auf der Linie. ⚪ „keine Meldung“ gibt es nur, wenn Beobachter an Bord sind.
- Neues Ereignis `ANWESEND` (Heartbeat beim Check-in und pro Halt): rotierendes Token statt Geräte-ID, wird bei Fahrtende gelöscht. Angezeigt wird nur grob „Nutzer an Bord“ / „niemand“.
- Banner bei dünner Datenlage je Linie. Dazu ein optionaler Satz zum Grundrisiko aus den aggregierten Statistiken, ohne Farbe.
- Feste Texte: „Keine Meldung heißt nicht frei.“ · „Die App kann sich irren. Sicher ist nur ein gültiges Ticket.“

### Q3 Zugteile als Bezugsgröße für alle Züge
**Erweitert** 3 (Kernbegriffe), 4.4, 5.4 und das Datenmodell in 7. **Löst** S-03, S-09.

- Ein physischer Zug hat 1 bis 3 Teile. Zwischen den Einheiten gibt es während der Fahrt keinen Durchgang, Wechsel sind nur am Halt möglich. `zugteil` ∈ {`vorne`, `mitte`, `hinten`, null} gilt für alle Züge, nicht nur für die Flügelung. `mitte` gibt es nur bei S3-Langzügen.
- Bei Flügelzügen vor Ohlsdorf ergibt sich der Teil aus dem Ziel beim Check-in (Airport = vorne). Positive Meldungen ohne Teil gelten für alle Teile. Negative Meldungen gelten nur für den eigenen Teil oder abgeschwächt (Q1).
- Die Zuglänge steht nicht im GTFS. Teilwahl wird nur angeboten, wo sie sicher existiert (S1-Flügelzüge, S3 werktags 5–19 Uhr) oder der Nutzer sie selbst angibt.

### Q4 Ausbreitung an Halten, Knoten und Endstationen
**Erweitert** 5.1 (`AUSGESTIEGEN_IN`) und die Darstellung in 4.2/4.3. **Löst** S-04, S-08, S-13 (hilft bei S-01, S-06, S-10).

- Der Anteil, der an Halt h den Zug verlässt (Δ = p · (1 − s(h))), wird verteilt: die Hälfte auf die Station h als Bahnsteighinweis (Halbwertszeit 10 min), die andere Hälfte auf die in den nächsten 10 Minuten (an Endstationen und in Neugraben 15 Minuten) abfahrenden Züge aller Linien und beider Richtungen.
- Angezeigt wird nur, was ≥ 0,15 erreicht: gestrichelt orange „möglich (Umstieg)“. **Aus Ausbreitung entsteht nie 🔴.** Pro Ursprungsmeldung höchstens 4 Hinweise, damit das Netz nicht flächig orange wird.
- Die Ausbreitung wird nur auf dem Gerät berechnet und nicht gespeichert. Meldungen werden nicht zu Spuren verknüpft (keine Bewegungsprofile, KONZEPT 8).

### Q5 Aktiv nachfragen statt auf Entwarnungen warten
**Erweitert** 4.5, 5.1 und 6.5. **Löst** S-01, S-06, S-12, S-13 (hilft überall).

- Neue Ereignistypen: `KEINE_GESEHEN`, `KONTROLLIERT_GERADE`, `FEHLALARM`, `ZUG_ENDET_HIER`, `ANWESEND`, `KORREKTUR`.
- Ausstiegsdialog für Melder: „Kontrolleure noch im Zug? [noch drin] [auch raus] [weiß nicht]“.
- Eingecheckte Nutzer in einem Zug oder Teil mit p ≥ 0,25 bekommen ein Banner in der Fahrt-Leiste: „Kontrolleure in deinem Zugteil? [Ja] [Nein] [Nur Sicherheit/Polizei] [weiß nicht]“. Höchstens 1 Frage pro Halt und 3 pro Fahrt. Wer nicht antwortet, liefert keine Information. Bei Bahnsteigmeldungen gibt es die gleiche Frage für Aus- und Durchsteigende.
- Antworten auf Rückfragen zählen nicht gegen das 10-Minuten-Limit (5.3 Nr. 3), sind aber auf 1 pro Halt begrenzt. Rückfragen kommen ohne Push-Token (Phase 3 bleibt optional), nur in der App bzw. als lokale Benachrichtigung während des Check-ins.

### Q6 Zuordnung über Reihenfolge und Kandidaten statt Fahrplanuhr
**Erweitert** 6.3, 6.4 und das Datenmodell in 7. **Löst** S-07, S-14 (hilft bei S-13).

- Überholverbot als Invariante: Auf einem Gleis behalten S-Bahnen im Regelbetrieb ihre Reihenfolge. Eine Crowd-Verspätung wird mindestens im Takt an die Folgezüge vererbt. Ein Zug verlässt die Abfahrtstafel erst nach einem Abfahrtssignal oder dem Ablauf seiner geschätzten Verspätung.
- Bahnsteigmeldungen zu einem Zug: Auswahl aus „gerade da / vor ≤ 3 min abgefahren“ mit Linie, **Ziel** und Zeit. Ist die Wahl unklar, gibt es einen Kandidaten-Split.
- Check-in-Korrektur mit Neubindung der eigenen Meldungen (`KORREKTUR`).
- Datenmodell, Ergänzung zu 7:
  ```sql
  alter table meldungen
    add column kandidaten  jsonb,   -- [{"fahrt_key": "...", "w": 0.6}, …] bei unklarer Zuordnung
    add column bezug_id    uuid,    -- Antwort/Korrektur bezieht sich auf diese Meldung
    add column quelle      text,    -- 'checkin' | 'bahnsteig' | 'rueckfrage' | 'telegram'
    add column halt_index  int;     -- Halt der Fahrt, an dem beobachtet wurde (Basis für Q1)
  -- zugteil: zusätzlich 'mitte'; typ: zusätzlich KEINE_GESEHEN, KONTROLLIERT_GERADE,
  -- FEHLALARM, ZUG_ENDET_HIER, ANWESEND (ohne geraet, mit rotierendem Token), KORREKTUR
  ```

**Bilanz:** 3 × ✅ (S-03, S-05, S-12, jeweils im Rahmen des Erkennbaren), 9 × 🟡 (S-01, S-04, S-06, S-07, S-08, S-09, S-10, S-13, S-14), 2 × ❌ im Kern (S-02, S-11). Die beiden ❌-Fälle sind dieselbe Grenze: Was niemand sieht, kann die App nicht wissen. Sie kann es nur ehrlich anzeigen (Q2).
