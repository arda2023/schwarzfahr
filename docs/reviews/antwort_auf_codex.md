# Antwort auf das Codex-Review und Vorschlag für Version 3

Stand: 3. November 2026. Bezieht sich auf [`codex_review_konzept_v2.md`](codex_review_konzept_v2.md), [`../alternativen.md`](../alternativen.md), [`../recherche/echtzeitdaten.md`](../recherche/echtzeitdaten.md) und [`../recherche/recht_foto_verifikation.md`](../recherche/recht_foto_verifikation.md).

---

## 1. Wo Codex recht hat

| Punkt | Bewertung |
|---|---|
| **Wahrscheinlichkeit 1,275** (0,85 × Gewicht 1,5) | Stimmt, das ist ein Fehler in KONZEPT 5.3. Gewichte dürfen nicht die Startwahrscheinlichkeit multiplizieren. Für den Start sind feste, nachvollziehbare Zustände besser, wie Codex vorschlägt. |
| **Anwesenheit ist keine Bestätigung** | Stimmt. KONZEPT 5.3 zählt „Beobachter an Bord“ als frische Sichtung. Ein Handy in der Tasche sieht nichts. Abgebrochene Lebenszeichen heißen „unbekannt“, nicht „ausgestiegen“. |
| **Getrennte Sicherheit für Beobachtung und Zuordnung** | Stimmt, und das ist der wichtigste Punkt des Reviews. Jemand kann sicher Kontrolleure sehen und trotzdem den falschen Zug wählen. Das Unsicherheitsband (Abschnitt 3) macht genau diese Trennung. |
| **„Neue Warnung gewinnt immer“** | Stimmt teilweise. Das Rate-Limit begrenzt den Schaden, aber Wiederholungen derselben Quelle dürfen eine Warnung nicht verlängern. |
| **Teamverfolgung streichen** | Einverstanden. Viel Aufwand, viele Fehlzuordnungen, Datenschutzrisiko. |
| **Server berechnet den Status** | Einverstanden. Ich hatte in v2 die Berechnung auf dem Handy gewählt, vor allem wegen der Teams. Ohne Teams spricht alles für den Server: gleicher Stand überall, Regeln ändern ohne App-Update, keine Einzelmeldungen im Umlauf. |
| **Web braucht eigenen Vertrauensweg** | Stimmt. Wenn die App aus den Stores fliegt, ist das Web der Hauptweg. Dort kann Vertrauen nur aus Kontoalter und unabhängig bestätigten früheren Meldungen kommen. |
| **Verbindliche Regelversion** | Stimmt. Die Lösungsdateien widersprechen sich in Details (z. B. Löschfristen 2 h / 6 h / 24 h). Verbindlich ist nur `KONZEPT.md`; die Szenario-Dateien sind Arbeitsmaterial. Das sollte oben in den Dateien stehen. |
| **Zusätzliche Szenarien** | Gute Liste, vor allem: Rücknahme kommt vor der Meldung an, Lücken nach Verbindungsabbruch, Datenquelle liefert „erfolgreich“ alte Daten, echte Großkontrolle löst die Spam-Notbremse aus. |
| **Kosten** | Rechnung geprüft: 28,8 Mio. Nachrichten kosten bei Supabase Pro etwa 95 $/Monat. Nur Änderungen senden etwa 35 $, Verspätungen als CDN-Snapshot etwa 25–35 $ (siehe Echtzeit-Recherche, ⚠️ Preise aus Zweitquellen). |
| **Zeitplan** | Stimmt. Meine Wochenangaben waren für eine Person zu optimistisch. |
| **„Gibt es nirgends“ ist eine Annahme** | Fair. Die Konkurrenzrecherche fand nichts Zugbezogenes, das ist aber kein Beweis. |

## 2. Wo ich Codex widerspreche oder ergänze

1. **Zwei Oberflächen (React fürs Web, Flutter für Handys) sind für eine Person zu viel.** Codex will den Umfang verkleinern und schlägt gleichzeitig zwei getrennte Apps vor, die beide gepflegt werden müssen. Besser eine Codebasis:
   - Start als **leichte Web-App** (React oder Preact mit TypeScript). Sie lädt auch am Bahnsteig mit schlechtem Netz schnell, was bei Flutter Web mit mehreren MB Startgröße schwieriger ist.
   - Später, wenn native Funktionen nötig sind (Geräteprüfung, Bewegungssensor), **dieselbe Web-App mit Capacitor** als iOS- und Android-App verpacken. Erst wenn das nicht reicht, über Flutter nachdenken.
2. **Codex nennt die Zugzuordnung das zentrale ungelöste Risiko, behält sie aber als Kern.** Echtzeit allein löst das nicht, das sagt Codex selbst. Die bessere Antwort ist, die App so zu bauen, dass sie den genauen Zug gar nicht kennen muss (Abschnitt 3).
3. **Phase 1 von Codex (Web-Prototyp mit simulierten Meldungen) testet das falsche Risiko.** Ob Leute die Anzeige verstehen, ist wichtig. Das größere Risiko ist aber, ob in Hamburg überhaupt genug gemeldet wird: Die Telegram-Gruppe „Kontrollettis HH“ hat 627 Mitglieder und 8 Nachrichten in 30 Tagen. Das sollte zuerst gemessen werden.
4. **Fahrt-ID:** Codex' Vorschlag (Quelle + Datensatzversion + GTFS-Fahrt-ID + Betriebstag) ändert sich bei jedem monatlichen Datensatz. Man braucht so oder so eine Zuordnung zwischen Versionen. Mit dem Band-Ansatz ist das kaum noch wichtig, weil die Wahrheit der Anker (Ort, Zeit, Richtung) ist und nicht die Fahrt.
5. **Kopplungen:** „Gleiche Zeiten beweisen keine Kopplung“ stimmt grundsätzlich. Für die S1 in Ohlsdorf ist die Flügelung aber dokumentiert und im GTFS 82-mal pro Tag sichtbar. Als Daten mit „vor Ort bestätigt: ja/nein“ pflegen reicht.
6. **Großkontrolle 90 min:** Codex hat recht, dass eine feste Mindestdauer zu langen Fehlwarnungen führt. Besser: langsames Abklingen, das jede neue Meldung erneuert.

## 3. Vorschlag: „Stationen + Band“ statt „jeder Zug einzeln“

Ausführlich in [`../alternativen.md`](../alternativen.md). Kurz:

### Das Unsicherheitsband
Eine Zugmeldung ist nur ein Anker: **Station, Uhrzeit, Richtung** (Linie, falls bekannt). Die App rechnet daraus, wo die Kontrolleure jetzt sein können, und zeichnet das als Band auf dem Streifen:

```
erwartete Zeit an Station S = Ankerzeit + Fahrzeit laut Fahrplan
Breite = ± (1 min + 0,15 min pro Halt)
```

- **Verspätungen spielen kaum eine Rolle,** weil die Ankerzeit eine echte Uhrzeit ist. Ist der Zug 7 min zu spät, steckt das schon drin.
- **Am Bahnsteig reicht „der Zug, der jetzt einfährt“.** Liegt er im Band, warnt die App.
- **Fehler fallen breit statt falsch aus.** Ist unklar, welcher von zwei Zügen gemeint ist, warnt die App beide als „möglich“. In v2 hätte sie einen geraten und der richtige hätte „keine Meldung“ gezeigt. Das war der gefährlichste Fehler.
- **Messbar:** Kommt stromabwärts eine zweite Meldung, lag sie im Band oder nicht? Diese Trefferquote zeigt, wie gut die App ist.
- Beispiel S1 in der HVZ: Meldung in Blankenese 08:21, in Altona erwartet 08:35 ± 1,9 min. Meist liegt **ein** Zug im Band, selten zwei.

### Stationen als Hauptprodukt
Kontrollen am Bahnsteig, am Ausgang und Großkontrollen sind ortsfest, dauern lange und werden von vielen gesehen. Dort entstehen von selbst viele unabhängige Meldungen, also echtes Vertrauen. Eine Stationsmeldung gilt an Knoten für alle Linien.

### Stufen
1. **Zuerst (wenige Wochen):** Web-App und Telegram-Bot mit Knöpfen, ohne Freitext. Meldungen: Bahnsteig, Ausgang, Großkontrolle, „im Zug“ als Anker. Anzeige: Stationen, Abschnitte, „meine nächsten Züge ab hier“ mit Band. Keine Echtzeit nötig. **Messen:** Meldungen pro Tag, Anteil „im Zug“, Zeit bis zur zweiten Meldung, Band-Trefferquote.
2. **Wenn die Zahlen tragen** (z. B. ≥ 20 Meldungen pro Tag): App mit Linienstreifen und Band, Echtzeit für die Abfahrtszeiten, optionaler Anker-Check-in für Rückfragen, Haltezähler über den Bewegungssensor im Tunnel.
3. **Nur mit Messdaten:** die fahrtgenaue Zugkarte aus v2, wenn das Band im Median höchstens einen Zug breit ist und die Trefferquote bei mindestens 80 % liegt.

**Nie:** Bluetooth-Erkennung der Mitreisenden (der Prüfdienst könnte mit jeder Scanner-App sehen, wer die App benutzt), Foto-Upload, GPS-Verweildauer als Vertrauensbonus, Teamverfolgung.

## 4. Deine Fragen

### Grok soll regelmäßig die hvv-App prüfen?
**Nicht nötig, und 1–3 Mal am Tag bringt für Verspätungen nichts.** Details: [`../recherche/echtzeitdaten.md`](../recherche/echtzeitdaten.md).
- Verspätungen ändern sich minütlich. Um Züge im 5-Minuten-Takt zu trennen, bräuchte man an etwa 12 Stützstationen jede Minute einen Abruf, im City-Tunnel alle 30 s. Das sind 17.000 bis 43.000 Abrufe am Tag.
- Ein Sprachmodell ist dafür das falsche Werkzeug: teuer, langsam, und es kann Zahlen falsch ablesen. Die hvv-App bedienen kann es nicht.
- Das massenhafte Auslesen der hvv-Daten ist rechtlich angreifbar (Zweckbindung der hvv-Schnittstelle, § 87b UrhG zum Datenbankschutz).
- **Besser und kostenlos:** DB Timetables API, 60 Abrufe pro Minute, mit S-Bahn-Hamburg-Echtzeit. Etwa 14 Stützstationen (Wedel, Blankenese, Altona, Hbf, Berliner Tor, Barmbek, Ohlsdorf, Poppenbüttel, Airport, Eidelstedt, Harburg, Neugraben, Bergedorf, Buxtehude) alle 20 s abfragen. Jede Station liefert die Prognose für alle Züge, die sie noch anfahren; man muss also nicht jede Station abfragen. Dazu den DELFI-Echtzeitfeed testen.
- **Mit dem Band-Ansatz braucht die Warnfunktion gar keine Echtzeit.** Echtzeit verbessert dann nur die Anzeige „Zug kommt in 6 min“.
- Sinnvoller Einsatz für Grok: einmal am Tag Baustellen- und Störungsmeldungen zusammenfassen, die du dann selbst prüfst.

### Massenkontrollen an Bahnhöfen melden
**Ja, unbedingt.** Das wird im Vorschlag sogar zum Hauptprodukt (siehe 3).

### Vertrauen über GPS-Verweildauer
Die Idee ist verständlich, funktioniert aber gerade dort schlecht, wo sie gebraucht wird:
- **Die S-Bahn-Bahnsteige am Hbf (City-Tunnel), am Jungfernstieg und in Altona liegen unter der Erde.** Dort gibt es kein GPS, das Handy schätzt über WLAN und Funkzellen auf 30–500 m. Bahnsteig, Zwischenebene und Straße sind nicht unterscheidbar.
- **Stehen beweist keine Schlange.** Es stehen auch Wartende, Personal, Security und die Kontrolleure selbst. Wer einer Kontrolle ausweicht, geht eher weg.
- **GPS lässt sich fälschen**, auf Android und im Browser sehr leicht.
- **Datenschutz:** Minutenlanges Tracking genau am Ort einer Kontrolle wäre ein Beleg „Gerät X war dort“.

Brauchbar ist nur eine **Plausibilitätsprüfung auf dem Handy**: War das Handy in den letzten 15 min in der Nähe der Station? Gesendet wird nur „plausibel / unbekannt / unplausibel“, keine Koordinaten. „Unbekannt“ (Tunnel) darf nicht bestraft werden.

**Besseres Vertrauen bei Massenkontrollen:** Gerade dort sehen viele Leute die Kontrolle. Mehrere unabhängige Meldungen von verschiedenen Konten innerhalb weniger Minuten sind schneller, billiger und schwerer zu fälschen als jede Ortsprüfung.

### Selfie als Beweis
**Praktisch rate ich ab** (Details in [`../alternativen.md`](../alternativen.md), Abschnitt 2.8):
- **Wer prüft das Bild?** Menschen schaffen das nicht rund um die Uhr innerhalb von Minuten. Eine KI erkennt „viele Menschen im Bahnhof“, aber nicht „das sind Kontrolleure“, zumal der Prüfdienst oft in Zivil arbeitet.
- **Fälschbar:** Foto eines Bildschirms oder einer beliebigen Menschenmenge.
- **Widerspruch:** Das eigene Gesicht soll beweisen, dass das Foto live ist, wird aber verpixelt.
- **Konflikt:** Wer Kontrolleure sichtbar fotografiert, wird eher angesprochen. Das trifft oft genau die Person ohne Ticket.
- **Gefahr für den Melder:** Foto + Ort + Zeit auf dem Server ist bei einem Datenleck oder einer Herausgabe ein Beweisstück.
- **Nutzen gering:** Zwei unabhängige Ein-Tap-Meldungen sind schneller und schwerer zu fälschen.

Zur Rechtslage: siehe Abschnitt 5.

## 5. Rechtslage zum Selfie

Ausführlich mit Normtexten und Urteilen: [`../recherche/recht_foto_verifikation.md`](../recherche/recht_foto_verifikation.md). Keine Rechtsberatung.

> **Quellenhinweis:** Die Primärquellen (gesetze-im-internet.de, eur-lex, dejure, curia, bahnhof.de, hvv.de) waren aus der Rechercheumgebung gesperrt. Die Normtexte entsprechen der amtlichen Fassung, wurden aber nicht live abgeglichen. Urteile stammen aus Suchauszügen, teils aus Sekundärquellen. **Vor einer Entscheidung alle Aktenzeichen selbst im Original prüfen** (oder von einer Anwältin prüfen lassen).

**Die These „Ich fotografiere mich selbst, also dürfen Fremde mit drauf sein“ hat im Gesetz keine Grundlage.** Was dort steht:

1. **Aufnehmen:** Das Kunsturhebergesetz (KUG) regelt nur das Verbreiten und öffentliche Zeigen (§ 22 S. 1 KUG). Schon das Anfertigen eines Fotos kann aber das allgemeine Persönlichkeitsrecht verletzen (Art. 2 Abs. 1 i. V. m. Art. 1 Abs. 1 GG, § 823 Abs. 1, § 1004 BGB analog), auch im öffentlichen Raum. Ob es rechtswidrig ist, entscheidet eine Abwägung im Einzelfall (BGH VI ZR 272/94).
2. **§ 201a StGB** (Bildaufnahmen aus dem höchstpersönlichen Lebensbereich) passt nicht: Er verlangt eine Wohnung, einen gegen Einblick geschützten Raum, Hilflosigkeit oder eine Aufnahme, die dem Ansehen erheblich schadet.
3. **Upload an den Betreiber = Verbreiten** im Sinne von § 22 KUG: Darunter fällt jede Weitergabe an Dritte, auch digital.
4. **„Beiwerk“** gibt es nach § 23 Abs. 1 Nr. 2 KUG nur neben einer **Landschaft oder Örtlichkeit**, nicht neben dem Fotografierenden. Kontrolleure, die der Zweck des Fotos sind, sind kein Beiwerk. Eine Schlange ist keine „Versammlung“ nach Nr. 3.
5. **DSGVO für den Melder:** Die Ausnahme für rein persönliche Tätigkeiten (Art. 2 Abs. 2 lit. c) legt der EuGH eng aus. Bei einem Upload an eine App ist sie fraglich.
6. **DSGVO für den Betreiber gilt immer.** Er braucht eine Rechtsgrundlage (praktisch Art. 6 Abs. 1 lit. f), muss Datensparsamkeit und Datenschutz durch Technik umsetzen (Art. 5, Art. 25) und die abgebildeten Personen informieren (Art. 13/14).
7. **Verhältnis KUG zu DSGVO** ist außerhalb des Journalismus nicht höchstrichterlich geklärt. Das ist ein echter Streit in der Rechtsprechung und Literatur, keine Vermutung.
8. **DB-Hinweise zum Fotografieren (2024), wörtlich:** „Für Nahaufnahmen von Personen müssen Sie vorab die Einwilligung der abgebildeten Personen einholen.“ Und: Nicht gestattet sind „Nahaufnahmen von DB-Mitarbeitenden ohne deren vorherige Zustimmung“. Den Wortlaut der aktuellen DB-InfraGO-Hausordnung und eine Foto-Regel des hvv konnte die Recherche nicht belegen.
9. **Alle Gesichter auf dem Gerät verpixeln und das Bild sofort löschen** senkt die Risiken deutlich. Es reicht aber nicht, wenn Personen über andere Merkmale (Kleidung, Statur, Ort und Zeit) erkennbar bleiben.

**Ergebnis:** Rechtlich und praktisch spricht alles gegen die Selfie-Funktion. Sie widerspricht außerdem der Regel „keine Fotos“ in KONZEPT 8, die genau aus diesen Gründen dort steht. Vertrauen bei Massenkontrollen entsteht besser durch mehrere unabhängige Meldungen.
