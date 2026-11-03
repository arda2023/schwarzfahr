# Konzept: Kontroll-Melder für die S-Bahn Hamburg

Stand: 3. November 2026 · Arbeitstitel (intern): `schwarzfahr`
Grundlage: [`recherche/sbahn_hamburg_recherche.md`](recherche/sbahn_hamburg_recherche.md) und [`recherche/konkurrenz_kontrolleur_apps.md`](recherche/konkurrenz_kontrolleur_apps.md)

---

## 1. Die Idee in einem Satz

Eine App zeigt jede S-Bahn-Linie als senkrechten Stationsstreifen. Darauf sieht man **jeden fahrenden Zug einzeln**, und Fahrgäste melden Kontrollen **an einem bestimmten Zug** oder **an einer Station**. Die Meldung fährt mit dem Zug mit, bis jemand meldet, dass die Kontrolleure ausgestiegen sind.

Das gibt es so noch nirgends. Alle Konkurrenten (FreiFahren, KontrollRadar, schwarzkappler.info) melden nur „Station X, Linie Y“ oder setzen einen Punkt auf eine Karte. Siehe Konkurrenzanalyse, Abschnitt „Lücken“.

---

## 2. Kurze Antwort: Brauche ich eine API?

| Was | Braucht man eine fremde API? | Lösung |
|---|---|---|
| Stationen, Linienverlauf, Abfahrtszeiten | **Nein** | Offener hvv-GTFS-Datensatz (Download, Lizenz dl-de/by-2.0). Daraus erzeugt ein Skript einmal pro Monat eine kleine JSON-Datei, die die App lädt. |
| Wo ist welcher Zug gerade? | **Nein** (für den Anfang) | Aus dem Fahrplan berechnen: Zug fährt 08:12 ab Wedel, jetzt ist 08:24 → er ist zwischen Iserbrook und Blankenese. Passiert komplett auf dem Handy. |
| Welche Station ist der User? | **Nein** | Eintippen, Favorit oder GPS (nächste Station aus den GTFS-Koordinaten). |
| Meldungen von User A an User B | **Ja, aber deine eigene** | Ohne Server geht das nicht. Ein kleines Backend (z. B. Supabase) speichert Meldungen und verteilt sie live an alle. |
| Verspätungen | **Optional, später** | Phase 1: Nutzer sind die Echtzeitquelle (siehe 6.4). Phase 3: DB Timetables API (kostenlos) über einen eigenen Cache-Server. |

**Fazit:** Für den Fahrplan brauchst du keine Schnittstelle. Du brauchst nur einen eigenen kleinen Server für die Meldungen, und den bekommst du bei Supabase oder Firebase für den Start kostenlos.

---

## 3. Kernbegriffe

| Begriff | Bedeutung | Beispiel |
|---|---|---|
| **Linie / Richtung** | Linie und Fahrtrichtung | S1 Richtung Airport/Poppenbüttel |
| **Streifen** | Die geordnete Stationsliste einer Linie und Richtung, ggf. mit Gabelung | Wedel … Ohlsdorf, dann Gabel Airport / Poppenbüttel |
| **Fahrt** | Ein konkreter Zug an einem konkreten Tag | „S1, 10.11.2026, 08:12 ab Wedel“ |
| **Physischer Zug** | Was tatsächlich auf dem Gleis fährt. Bei der S1 sind bis Ohlsdorf **zwei Fahrten ein Zug** (Flügelung). | Airport-Fahrt + Poppenbüttel-Fahrt |
| **Zugmeldung** | „Kontrolleure sind in diesem Zug“ | wandert mit dem Zug |
| **Stationsmeldung** | „Kontrolleure stehen am Bahnsteig / Ausgang“ | gilt für alle Züge und alle Linien an der Station |

### Stabile Fahrt-ID

Die `trip_id` im GTFS ändert sich bei jedem neuen Datensatz. Meldungen brauchen aber eine ID, die Server und alle Handys gleich berechnen. Deshalb:

```
fahrtKey = "<Linie>|<Startstation>|<Datum>|<Abfahrt am Start>"
Beispiel:  "S1|Wedel|2026-11-10|08:12"
```

Für Flügelzüge bekommt der physische Zug die ID der Airport-Fahrt, die Poppenbüttel-Fahrt verweist darauf (`partnerOf`).

---

## 4. Screens

### 4.1 Start: Abfahrtstafel mit Kontrollstatus

Das ist der wichtigste Screen. Er beantwortet in 2 Sekunden: „Welcher Zug kommt, und ist da etwas gemeldet?“

```
┌──────────────────────────────────┐
│ 📍 Altona               (GPS) ▾ │
│──────────────────────────────────│
│ S1 → Airport/Poppenb.            │
│   09:42  in 3 min   ⚪ keine Meld.│
│   09:52  in 13 min  🔴 im Zug    │  ← Meldung aus Blankenese
│   10:02  in 23 min  ⚪            │
│ S3 → Neugraben                   │
│   09:45  in 6 min   🟠 unsicher  │
│──────────────────────────────────│
│ ⚠ Station Altona: keine Meldung  │
│ ⚠ Jungfernstieg: Bahnsteig, 9 min│
│──────────────────────────────────│
│        [  Kontrolle melden  ]    │
└──────────────────────────────────┘
```

Tippen auf eine Abfahrt öffnet die Zugkarte (4.2).

### 4.2 Zugkarte: der Streifen, zum Wischen

So wie in deiner Skizze. Jede Karte ist **eine Fahrt**. In der Mitte steht der nächste Zug ab deiner Station.

- **Nach links wischen:** der Zug davor, der schon an dir vorbei ist und weiter vorne fährt.
- **Nach rechts wischen:** der Zug danach, der noch kommt.

```
   ◀ 09:32 (vor dir)      09:42  S1 → Airport       09:52 ▶
┌──────────────────────────────────┐
│  S1 · ab Wedel 09:15             │
│                                  │
│  ○ Wedel                 09:15   │
│  ○ Rissen                09:19   │
│  ○ Sülldorf              09:22   │
│  ● Iserbrook   ◀ 🚆 hier  09:25  │  ← Zug gerade hier
│  ○ Blankenese  🔴         09:28   │  ← Station rot: Bahnsteigkontrolle
│  ○ Hochkamp              09:31   │
│  …                               │
│  ◎ Altona      (du)      09:42   │
│  …                               │
│  ○ Ohlsdorf  ─┬─ Airport  10:21  │
│               └─ Poppenbüttel    │
│                                  │
│  Zug: ⚪ keine Meldung            │
│  [ Ich bin in diesem Zug ]       │
└──────────────────────────────────┘
```

Darstellung auf der Karte:
- **Zug-Symbol** an der berechneten Position, mit Farbe für den Zugstatus.
- **Station rot** bei Stationsmeldung (gilt linienübergreifend).
- **Station mit rotem Rand** dort, wo eine Zugmeldung gemacht wurde („Kontrolleure eingestiegen in Blankenese“). So entsteht genau dein Beispiel: Für den Wedel-User leuchtet Blankenese auf der Karte des vorherigen Zuges rot.

### 4.3 Linienübersicht (Ergänzung zu deiner Idee)

Eine einzige Streifen-Ansicht pro Linie und Richtung, auf der **alle Züge gleichzeitig als Punkte** sitzen. Für den User in Altona ist das schneller als Wischen: Er sieht sofort, dass der Zug, der gerade bei Blankenese ist, rot ist.

```
S1 → Airport/Poppenbüttel
  Wedel        ○
  Rissen       ○   🚆⚪
  …
  Blankenese   ○   🚆🔴   ← dieser Zug hat Kontrolleure
  …
  Altona       ◎ du
  …
  Hbf          ○   🚆⚪
```

Bei 5-Minuten-Takt sind das bis zu 12 Punkte pro Richtung, das passt auf einen Bildschirm.

### 4.4 Melden: maximal 2 Taps

```
Kontrolle melden
  ( ) Im Zug         → Zug wird vorgeschlagen (Check-in oder GPS), 1 Tap bestätigen
  ( ) Am Bahnsteig   → Station wird vorgeschlagen (GPS), 1 Tap bestätigen
Optional: vorderer / hinterer Zugteil (nur S1 vor Ohlsdorf wichtig)
```

**Kein Freitext, keine Fotos, keine Beschreibung von Personen.** Das ist rechtlich wichtig (Persönlichkeitsrecht, DSGVO) und hält die Daten sauber.

### 4.5 Während der Fahrt

Wer eingecheckt ist, sieht oben eine Leiste mit seinem Zug und drei großen Buttons:

```
🚆 S1 09:42 ab Altona · jetzt: Königstraße
[ Kontrolleure hier ]  [ Sie sind ausgestiegen ]  [ Ich bin ausgestiegen ]
```

---

## 5. Meldungslogik (das Herzstück)

### 5.1 Ereignistypen

| Typ | Bezug | Wirkung |
|---|---|---|
| `IM_ZUG` | Fahrt + Station, an der es bemerkt wurde | Zug wird rot |
| `AN_STATION` | Station | Station wird rot (alle Linien) |
| `AUSGESTIEGEN_IN` | Fahrt + Station | Zug wird wieder neutral, Station wird rot (Bahnsteig) |
| `BESTÄTIGT` | Fahrt oder Station | Vertrauen steigt, Timer wird zurückgesetzt |
| `NICHT_MEHR_DA` | Fahrt oder Station | Vertrauen sinkt (löscht nicht, siehe 5.3) |
| `MELDER_RAUS` | Fahrt | Der Melder hat den Zug verlassen; der Zug wird „unsicher“ |

### 5.2 Zugstatus

```
             IM_ZUG
  ⚪ keine ────────────▶ 🔴 bestätigt ◀──── BESTÄTIGT
  Meldung                  │    │
     ▲                     │    │ MELDER_RAUS oder 15 min ohne Bestätigung
     │ AUSGESTIEGEN_IN     │    ▼
     │ oder Fahrt endet    │  🟠 unsicher ──(30 min / Fahrtende)──▶ ⚪
     └─────────────────────┘
```

- **⚪ heißt „keine Meldung“, nicht „sicher“.** Die App zeigt nie Grün. Fehlende Meldungen beweisen nichts, und die App soll nichts versprechen, was sie nicht weiß.
- Zugmeldungen enden spätestens, wenn die Fahrt an der Endstation ankommt.
- Stationsmeldungen verblassen nach 20 Minuten und verschwinden nach 45 Minuten (Werte aus der Konkurrenz: 20 bis 60 min).

### 5.3 Missbrauch und Fake-Entwarnungen

Kontrolleure oder Trolle könnten falsche Meldungen oder falsche Entwarnungen schicken. Gegenmittel:

1. **Eine Entwarnung löscht nichts.** Sie senkt nur das Vertrauen und wird angezeigt („1× als vorbei gemeldet“). Das hat Barcelona (MemetroMap) genau so gelöst.
2. **Nähe-Check auf dem Gerät:** Zugmeldungen nur, wenn man eingecheckt ist. Stationsmeldungen nur, wenn GPS höchstens ca. 1,5 km entfernt ist (wie FreiFahren). Ohne GPS nur mit niedrigem Gewicht.
3. **Rate-Limit:** maximal 1 Meldung pro 10 Minuten pro Gerät, serverseitig erzwungen.
4. **Vertrauenswert pro Gerät** (anonym): Meldungen, die später von anderen bestätigt werden, erhöhen ihn.
5. **Vertrauen einer Meldung** = Summe der Gewichte aller Bestätigungen minus Entwarnungen, über die Zeit abklingend. Erst ab einem Schwellwert gilt eine Meldung als 🔴, darunter 🟠.

### 5.4 Flügelung S1 in Ohlsdorf

- Bis Ohlsdorf zeigt die Karte **einen Zug mit zwei Zielen**. Eine Zugmeldung gilt für beide Teile.
- Hat der Melder „vorne“ oder „hinten“ angegeben, geht die Meldung ab Ohlsdorf nur an den richtigen Teil weiter: **vorne → Airport, hinten → Poppenbüttel**.
- Ohne Angabe erben beide Teile die Meldung als 🟠 unsicher.
- In der Gegenrichtung werden beide Teile in Ohlsdorf vereinigt, die Meldungen beider Teile gelten dann für den ganzen Zug.

Flügelpaare erkennt das Datenskript automatisch: zwei S1-Fahrten mit identischen Zeiten bis Ohlsdorf.

---

## 6. Wie die App Züge kennt, ohne API

### 6.1 Datenpipeline (einmal pro Monat)

```
hvv-GTFS (ZIP, ~alle Linien)
        │  Skript (Dart oder Python), läuft als GitHub Action monatlich
        ▼
  1. nur S-Bahn-Linien behalten (route_short_name S1, S2, S3, S5, S7 …)
  2. Stationen zusammenfassen (parent_station), Koordinaten behalten
  3. Streifen je Linie und Richtung bauen (häufigstes Haltemuster + Äste)
  4. Fahrten mit Haltezeiten und Verkehrstagen (calendar / calendar_dates)
  5. Flügelpaare S1 erkennen, S5-Teilfahrten in Neugraben zusammenfügen
        ▼
  sbahn-hh.json  (geschätzt einige 100 KB gzip)
        ▼
  GitHub Pages / CDN  →  App lädt beim Start, wenn eine neue Version da ist
```

Wichtig aus der Recherche:
- **Linien nie hart codieren.** Im Dezember 2026 kommt wahrscheinlich die S2 nach Altona zurück und die S7 fällt weg. Das Skript erzeugt alles neu.
- Die S7 ist eine Schleife mit unterschiedlichen Stationen je Richtung. Ein Streifen pro Richtung löst das automatisch.
- Namensnennung „Hamburger Verkehrsverbund GmbH“ im Impressum bzw. in der App ist Pflicht (dl-de/by-2.0).

### 6.2 Zugposition berechnen (auf dem Handy)

```dart
// Pseudocode
Position positionVon(Fahrt f, DateTime jetzt, {Duration verspaetung = Duration.zero}) {
  final t = jetzt.subtract(verspaetung);
  for (var i = 0; i < f.halte.length - 1; i++) {
    final a = f.halte[i], b = f.halte[i + 1];
    if (t.isBefore(a.abfahrt)) return Position.amHalt(a);          // steht noch
    if (t.isBefore(b.ankunft)) {
      final anteil = (t - a.abfahrt) / (b.ankunft - a.abfahrt);
      return Position.zwischen(a, b, anteil);                       // fährt
    }
  }
  return Position.beendet();
}
```

Die Reihenfolge der Karten zum Wischen ergibt sich aus der Abfahrtszeit am Start: Die vorherige Fahrt ist der Zug vor dir.

### 6.3 Welcher Zug ist meiner?

1. **Beim Einsteigen:** Die Abfahrtstafel zeigt die nächsten Züge, der User tippt „Ich steige ein“. Fertig.
2. **Schon im Zug:** Die App nimmt GPS-Position und Bewegungsrichtung und schlägt die Fahrt vor, deren berechnete Position am nächsten liegt („Bist du in der S1 von 09:15 ab Wedel?“). Der User bestätigt mit einem Tap.
3. **Ohne GPS:** Linie, Richtung und Station manuell auswählen, die App nimmt den nächsten Zug.

### 6.4 Verspätungen: die Nutzer sind die Echtzeit-API

Das ist der Haken, den Grok richtig benannt hat: Bei 5-Minuten-Takt und 6 Minuten Verspätung landet eine Meldung beim falschen Zug. Lösung in drei Stufen:

1. **Check-in statt Raten.** Wer eingecheckt ist, meldet immer am richtigen Zug, egal wie spät er ist. Die Fahrt-ID kommt aus dem Check-in, nicht aus der Uhrzeit.
2. **Crowd-Verspätung.** Ein eingecheckter User erreicht laut GPS Rissen um 09:22 statt 09:19. Das Handy schickt nur „Fahrt X, Rissen, +3 min“ an den Server (keine GPS-Rohdaten). Alle anderen Handys verschieben diesen Zug dann um 3 Minuten. Je mehr Nutzer, desto genauer.
3. **Später: DB Timetables API** (kostenlos, 60 Aufrufe/min, mit Zugnummern). Ein kleiner eigener Server fragt die wichtigsten Stationen reihum ab und verteilt die Verspätungen an alle Apps. Erst nötig, wenn die Zuordnung in der Praxis zu ungenau ist.

### 6.5 Aussteigen erkennen

Bei Erlaubnis des Users, nur solange er eingecheckt ist:

| Signal | Funktioniert im Tunnel? | Aussagekraft |
|---|---|---|
| **Bewegungserkennung des Handys** (iOS CoreMotion / Android Activity Recognition: „im Fahrzeug“ → „gehend“) | **ja** | hoch, sparsam im Akku |
| GPS: User bleibt an Station, Zug fährt laut Fahrplan weiter | nein (City-Tunnel, Harburger Tunnel) | hoch |
| Button „Ich bin ausgestiegen“ | ja | eindeutig |
| Fahrt hat Endstation erreicht | ja | automatisch |

Wird ein Ausstieg erkannt, sendet die App `MELDER_RAUS`, und sein Zug wird 🟠 unsicher, wenn er der einzige Melder war. Der Standort wird **nur auf dem Gerät** ausgewertet (wie bei FreiFahren).

---

## 7. Architektur

```
┌────────────────────────┐        ┌───────────────────────────┐
│ App (Flutter)          │        │ GitHub Pages / CDN        │
│ iOS · Android · Web    │◀───────│ sbahn-hh.json (monatlich) │
│                        │        └───────────────────────────┘
│ - Fahrplan lokal       │
│ - Zugpositionen lokal  │        ┌───────────────────────────┐
│ - Status-Berechnung    │◀──────▶│ Supabase                  │
│   lokal                │  live  │ - Tabelle meldungen       │
│ - GPS nur lokal        │        │ - Realtime (Push an alle) │
└────────────────────────┘        │ - Rate-Limit (Edge Func.) │
                                  │ - Löschjob nach 24 h      │
                                  └───────────────────────────┘
```

**Warum so:** Das Backend bleibt dumm und billig. Es speichert nur Meldungen und verteilt sie. Die ganze Logik (Position, Status, Flügelung) läuft auf dem Handy mit denselben Fahrplandaten. Das Meldungsvolumen ist winzig (Hamburg vielleicht einige Hundert pro Tag), jedes Handy kann einfach alle Meldungen der letzten 2 Stunden laden und neue live empfangen.

**Warum Flutter:** Eine Codebasis für iOS, Android **und Web**. Die Web-Version ist die Versicherung, falls Apple oder Google die App entfernen (FreiFahren 2026, Akha 2025).

### Datenmodell der Meldung

```sql
create table meldungen (
  id          uuid primary key default gen_random_uuid(),
  typ         text not null,          -- IM_ZUG, AN_STATION, AUSGESTIEGEN_IN, BESTÄTIGT, NICHT_MEHR_DA, MELDER_RAUS
  fahrt_key   text,                   -- "S1|Wedel|2026-11-10|08:12", null bei reiner Stationsmeldung
  station_id  text,                   -- GTFS parent_station
  zugteil     text,                   -- 'vorne' | 'hinten' | null
  verspaetung_sek int,                -- optional, Crowd-Verspätung
  geraet      text not null,          -- zufällige, anonyme Geräte-ID (kein Account)
  gewicht     real not null default 1,
  erstellt    timestamptz not null default now()
);
-- Löschjob: delete from meldungen where erstellt < now() - interval '24 hours';
```

Kein Name, keine E-Mail, keine GPS-Koordinaten, keine IP-Speicherung in eigenen Tabellen. Für Statistiken („wo wird am häufigsten kontrolliert“) nur aggregierte Zahlen pro Station und Wochentag behalten.

---

## 8. Recht, Store und Name

Keine Rechtsberatung. Zusammengefasst aus der Recherche:

- **In Deutschland gibt es kein Verbot** solcher Apps und kein bekanntes Urteil. Juristen diskutieren aber Unterlassungsansprüche und Beihilfe; die Lage ist ungeklärt. **Vor dem Launch: kurze Beratung bei einem Anwalt für IT-Recht.**
- **Name und Ton neutral halten.** „Schwarzfahr“ bleibt Repo-Name. Die App selbst sollte nicht so heißen und nicht zum Fahren ohne Ticket auffordern. Provokante Namen haben bei Akha Politik und Presse angeheizt, und ein solcher Name ist das größte Risiko bei der App-Store-Prüfung.
- **Keine Logos, keine offiziellen Netzpläne, nicht „S-Bahn Hamburg App“ oder „hvv“ im Namen** (Marke; CheckMyMetro wurde wegen Netzplänen angegangen). Eigene Streifengrafik ist unproblematisch.
- **Keine Fotos, kein Freitext, keine Personenbeschreibungen**, damit keine Bewegungsprofile einzelner Kontrolleure entstehen (DSGVO).
- **Impressumspflicht:** Wer die App anbietet, muss mit Name und Anschrift erreichbar sein. FreiFahren löst das über einen eingetragenen Verein. Überlege das, bevor du mit deinem Privatnamen online gehst.
- **Echter Zusatznutzen** (Abfahrtstafel, Verspätungen, Störungen) macht die App alltagstauglich. Die Store-Beschreibung muss aber zeigen, was die App tatsächlich tut. Wer dort etwas anderes beschreibt, verstößt selbst gegen die Store-Regeln.

---

## 9. Kaltstart: Woher kommen die ersten Meldungen?

Hamburg ist fast leer (Telegram „Kontrollettis HH“ ist inaktiv, KontrollRadar ohne bekannte Größe). Ohne Meldungen ist die App nutzlos. Deshalb:

1. **Nutzen ohne Meldungen:** Die Abfahrtstafel mit Zugpositionen funktioniert ab Tag 1 auch ohne einen einzigen Melder.
2. **Klein und dicht starten:** erst die S1 (am häufigsten genannte Linie in der Hamburger Facebook-Gruppe), dann alle S-Bahn-Linien. Lieber eine Linie mit vielen Meldungen als fünf leere.
3. **Telegram-Bot:** Meldungen per Bot entgegennehmen und in die App spiegeln (FreiFahren ist so gewachsen). Umgekehrt postet der Bot App-Meldungen in einen Kanal.
4. **Gezielter Launch:** ein Post in r/hamburg und Uni-Gruppen (PtvAlert hatte so 10.000 Downloads in wenigen Tagen).
5. **Keine Paywall vor Meldungen.** Bei PtvAlert sank dadurch die Zahl der Meldungen. Wenn Geld, dann Spenden oder ein „Unterstützer“-Abo ohne Funktionssperre.

---

## 10. Fahrplan der Umsetzung

### Phase 0 – Daten und Prototyp (1–2 Wochen)
- [ ] GTFS-Skript: S-Bahn filtern, Streifen und Fahrten als JSON erzeugen
- [ ] Flügelpaare S1 und S5-Teilfahrten automatisch erkennen, mit Tests gegen die Recherche-Zahlen (z. B. ~12 Züge je Richtung S1 in der HVZ)
- [ ] Flutter-Prototyp: Streifen zeichnen, Züge nach Fahrplan bewegen, Karten wischen

### Phase 1 – MVP (2–4 Wochen)
- [ ] Abfahrtstafel, Zugkarte, Linienübersicht
- [ ] Meldungen `IM_ZUG`, `AN_STATION`, `AUSGESTIEGEN_IN` über Supabase Realtime
- [ ] Anonyme Geräte-ID, Rate-Limit, Löschjob, Ablauf der Meldungen
- [ ] Web-Version + TestFlight / interner Android-Test mit Freunden auf der S1

### Phase 2 – Genauigkeit (2–4 Wochen)
- [ ] Check-in, Zugvorschlag per GPS
- [ ] Aussteigen erkennen (Bewegungserkennung + GPS + Button)
- [ ] Bestätigen / „nicht mehr da“, Vertrauenswerte, 🟠-Status
- [ ] Crowd-Verspätung
- [ ] Flügelung mit vorderem/hinterem Zugteil

### Phase 3 – Wachstum
- [ ] Verspätungen über DB Timetables API mit eigenem Cache-Server
- [ ] Push: „Kontrolle auf deiner Favoritenstrecke“ (optional, weil ein Push-Token die Anonymität schwächt)
- [ ] Telegram-Bot
- [ ] U-Bahn Hamburg (gleiches GTFS), danach andere Städte über bundesweite GTFS-Daten (gtfs.de / DELFI)

### Kosten zum Start
| Posten | Kosten |
|---|---|
| Supabase Free Tier | 0 € |
| GitHub Pages für die Fahrplan-JSON | 0 € |
| Apple Developer Program | 99 €/Jahr |
| Google Play | 25 € einmalig |
| Anwaltliche Erstberatung | grob 150–400 € (Schätzung) |

---

## 11. Offene Fragen

1. **Name der App** (neutral, ohne Marken).
2. **Allein oder als Verein?** Hat Folgen für Impressum und Haftung.
3. **Wie stark Tracking?** Bewegungserkennung ist für Aussteigen ideal, wirkt aber im Store-Review und bei Nutzern sensibel. Vorschlag: aus per Standard, beim ersten Check-in einmal fragen.
4. **Wie viel Vertrauen braucht 🔴?** Startwert: 1 Meldung eines eingecheckten Users reicht; ohne Check-in braucht es 2. Mit echten Daten nachjustieren.
5. **Was passiert im Dezember 2026?** Fahrplanwechsel abwarten und das Skript gegen den neuen Datensatz laufen lassen, bevor die Linien im MVP festgezurrt werden.
