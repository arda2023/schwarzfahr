# Konzept: Kontroll-Melder für die S-Bahn Hamburg

Version 2 · Stand: 3. November 2026 · Arbeitstitel (intern): `schwarzfahr`

Grundlage:
- [`recherche/sbahn_hamburg_recherche.md`](recherche/sbahn_hamburg_recherche.md) (Netz, Takte, Daten)
- [`recherche/konkurrenz_kontrolleur_apps.md`](recherche/konkurrenz_kontrolleur_apps.md) (Konkurrenz, Recht)
- [`szenarien/UEBERSICHT.md`](szenarien/UEBERSICHT.md) (67 fiktive Problem-Szenarien mit Lösungen)

**Was in Version 2 neu ist:** Die 67 Szenarien haben Lücken in Version 1 gezeigt. Eingearbeitet sind vor allem: ehrliche Anzeige mit Datenlage (5.2), Wahrscheinlichkeit pro Halt statt fester Timer (5.3), Entwarnungen zählen weniger als Warnungen (5.4), Meldung als Beobachtung mit Anker (5.1), Check-in über das Ziel am Zug (6.3), Zug, Fahrt und Kopplung getrennt (3), Server prüft und verteilt nur bereinigte Daten (7), Sicherheitsregeln (8) und Echtzeit schon vor dem öffentlichen Start (10). In eckigen Klammern stehen die Szenarien, aus denen eine Regel kommt, z. B. [S-01].

---

## 1. Die Idee in einem Satz

Eine App zeigt jede S-Bahn-Linie als senkrechten Stationsstreifen. Darauf sieht man **jeden fahrenden Zug einzeln**, und Fahrgäste melden Kontrollen **an einem bestimmten Zug** oder **an einer Station**. Die Meldung fährt mit dem Zug mit und wird an jedem Halt schwächer, bis sie jemand bestätigt oder entwarnt.

Das gibt es so noch nirgends. Alle Konkurrenten (FreiFahren, KontrollRadar, schwarzkappler.info) melden nur „Station X, Linie Y“ oder setzen einen Punkt auf eine Karte.

### Was die App nicht kann (und ehrlich sagen muss)

- **Was niemand sieht, kann niemand melden.** Still zugestiegene Kontrolleure und Kontrolleure in Zivil, die sich noch nicht ausgewiesen haben, sind unsichtbar [S-02, S-11, K-01].
- **Bei Großeinsätzen kommt oft keiner mehr zum Melden** [T-09].
- **Alles, was die App anzeigt, kann auch das Verkehrsunternehmen mitlesen** [M-05].

Darum gilt überall: **Keine Meldung heißt nicht „frei“.** Die App zeigt nie Grün und verwendet nie die Wörter „frei“ oder „sicher“.

---

## 2. Brauche ich eine API?

| Was | Fremde API nötig? | Lösung |
|---|---|---|
| Stationen, Linienverlauf, Abfahrtszeiten | **Nein** | Offener hvv-GTFS-Datensatz (dl-de/by-2.0). Ein Skript erzeugt daraus eine versionierte JSON-Datei (6.1). |
| Wo ist welcher Zug gerade? | **Nein** für einen Test, **ja** ab dem öffentlichen Start | Phase 1: aus dem Fahrplan plus Beobachtungen der Nutzer (6.2, 6.4). Ab Phase 2: Verspätungen und Ausfälle über die DB Timetables API (kostenlos) mit eigenem Cache-Server [Z-01, Z-02, Z-03]. |
| Welche Station ist der User? | **Nein** | Eintippen, Favorit oder GPS. |
| Meldungen von User A an User B | **Ja, deine eigene** | Kleines Backend (Supabase), das Meldungen prüft und bereinigt verteilt (7). |

**Fazit:** Für den Fahrplan brauchst du keine Schnittstelle. Für einen Test mit Freunden auf der S1 reicht der Fahrplan auch für die Zugpositionen. Vor dem öffentlichen Start brauchst du Echtzeitdaten, sonst landen Meldungen im dichten Takt beim falschen Zug. Messbarer Auslöser: Wenn in der HVZ mehr als 10 % der Check-ins später korrigiert werden müssen, ist es höchste Zeit.

---

## 3. Kernbegriffe

| Begriff | Bedeutung | Beispiel |
|---|---|---|
| **Linie / Richtung** | Linie und Fahrtrichtung | S1 Richtung Airport/Poppenbüttel |
| **Streifen** | Geordnete Stationsliste einer Linie und Richtung, ggf. mit Gabelung | Wedel … Ohlsdorf, dann Airport / Poppenbüttel |
| **Fahrt** | Eine GTFS-Fahrt an einem Betriebstag | S1, 10.11.2026, 08:12 ab Wedel nach Airport |
| **Kopplung** | Verbindung zwischen Fahrten: Flügelung, Vereinigung, Durchbindung, Wende | S1 Ohlsdorf, S5 Neugraben, S7 Altona |
| **Zugverband** | Kette von Fahrten, die über Kopplungen ein physischer Zug sind. Meldungen hängen am Verband. | Wedel→Ohlsdorf als Vollzug, danach zwei Teile |
| **Zugteil** | Einheit im Zug: `vorne`, `mitte` (nur S3-Langzug), `hinten`. Während der Fahrt gibt es zwischen den Einheiten keinen Durchgang, wechseln geht nur am Halt. | Airport-Teil = vorne ⚠️ vor Ort prüfen |
| **Zugmeldung** | „Kontrolleure sind in diesem Zug(teil)“ | wandert mit dem Zug |
| **Stationsmeldung** | „Kontrolleure stehen am Bahnsteig / Ausgang“ | gilt für alle Linien an der Station |
| **Team** | Hypothese „eine Gruppe bewegt sich“, nur auf dem Handy berechnet, nie gespeichert (5.5) | Team steigt in Altona von S1 in S3 um |
| **Bereich** | Pendeln zwischen Stationen oder Großkontrolle (5.5) | Großlage Hbf/Jungfernstieg |

### Stabile Fahrt-ID

In Version 1 hatten Airport- und Poppenbüttel-Fahrt eines Flügelpaares denselben Schlüssel. Das war ein Fehler [Z-04]. Neu:

```
fahrtKey = "<Linie>|<Start-DHID>|<Ziel-DHID>|<Betriebstag>|<Abfahrt am Start als GTFS-Zeit>"
Beispiel:  "S1|de:02000:<Wedel>|de:02000:<Airport>|2026-11-10|08:12"
```

- **Ziel im Schlüssel:** trennt die beiden Flügelfahrten.
- **DHID statt Name:** bundesweit stabil.
- **Betriebstag + GTFS-Zeit** (darf über 24:00 liegen): Nachtfahrten bleiben beim richtigen Betriebstag.

---

## 4. Screens

### 4.1 Start: Abfahrtstafel mit Kontrollstatus

```
┌──────────────────────────────────────┐
│ 📍 Altona                  (GPS) ▾   │
│ ● Live                                │  ← Verbindungsleiste (Live / Stand vor N min / Offline)
│──────────────────────────────────────│
│ S1 → Airport/Poppenb.                │
│   09:42  in 3 min  ⚪ keine Meldung · Nutzer an Bord
│   09:52  in 13 min 🔴 gemeldet · zuletzt Blankenese, vor 1 Halt
│   10:02  in 23 min ▒ keine Daten     │
│ S3 → Neugraben                       │
│   09:45  in 6 min  🟠 wahrscheinlich · Melder ausgestiegen
│──────────────────────────────────────│
│ Jungfernstieg: 🔴 Bahnsteig, vor 9 min│
│──────────────────────────────────────│
│ Keine Meldung heißt nicht frei.      │
│          [  Kontrolle melden  ]      │
└──────────────────────────────────────┘
```

### 4.2 Zugkarte: der Streifen, zum Wischen

Jede Karte ist ein Zugverband. In der Mitte steht der nächste Zug ab deiner Station.
- **Nach links wischen:** der Zug davor, der schon an dir vorbei ist.
- **Nach rechts wischen:** der Zug danach.

```
   ◀ 09:32 (vor dir)      09:42  S1 → Airport       09:52 ▶
┌──────────────────────────────────────┐
│  S1 · ab Wedel 09:15 · nach Fahrplan │  ← hohler Punkt = berechnet
│                                      │     gefüllter Punkt = beobachtet vor X min
│  ○ Wedel                 09:15       │
│  ○ Rissen   ┃ bemerkt     09:19      │  ← roter Rand: hier wurde die Meldung gemacht
│  ○ Sülldorf              09:22       │
│  ● Iserbrook  ◀ 🚆 hier  09:25       │
│  ○ Blankenese  🔴 Bahnsteig 09:28    │
│  …                                   │
│  ◎ Altona      (du)      09:42       │
│  …                                   │
│  ○ Ohlsdorf  ─┬─ vorne: Airport      │
│               └─ hinten: Poppenbüttel│
│                                      │
│  Zug: 🟠 wahrscheinlich              │
│  gemeldet 09:19 Rissen · 1× entwarnt │  ← Verlauf bleibt sichtbar [M-01]
│  Melder seit Blankenese nicht an Bord│
│  [ Ich bin eingestiegen ]            │
└──────────────────────────────────────┘
```

Die Ränder an Stationen heißen „bemerkt in“, nicht „eingestiegen in“ [K-01]. Jede Anzeige nennt „zuletzt bestätigt HH:MM bei Station“.

### 4.3 Linienübersicht

Ein Streifen pro Linie und Richtung, auf dem alle Züge gleichzeitig als Punkte sitzen. Für den User in Altona schneller als Wischen. Abschnitte ohne Beobachter sind schraffiert ▒ [T-02, K-10].

### 4.4 Melden: weiterhin 2 Taps, danach optionale Angaben

```
Kontrolle melden
  ( ) Im Zug         → eigener Check-in-Zug, 1 Tap bestätigen
  ( ) Am Bahnsteig   → Station per GPS, 1 Tap bestätigen
  ( ) Großkontrolle  → Station / Bereich
```

Der zweite Tap sendet sofort. Danach erscheinen 10 Sekunden lang optionale Chips, jeder ein Tap [K-01, K-02, K-04, K-08, K-11, K-12]:
- **Wo im Zug:** vorne / Mitte / hinten (nur angeboten, wo der Zugteil sicher existiert: S1 vor Ohlsdorf, S3-Langzug werktags 5–19 Uhr)
- **kontrollieren** / **nur gesehen**
- **sicher** / **unklar** (z. B. vielleicht Security oder Polizei)
- **waren schon vorher da** → Station wählen
- an der Station: **Ausgang**

Schutz gegen Fehlbedienung [T-08]: Der Melde-Button reagiert nicht bei ausgeschaltetem Bildschirm oder in der Tasche (Näherungssensor), und nach dem Senden gibt es 10 Sekunden „Rückgängig“.

**Kein Freitext, keine Fotos, keine Beschreibung von Personen.**

### 4.5 Während der Fahrt

```
🚆 S1 09:42 ab Altona → Airport · jetzt: Königstraße   [Station korrigieren]
[ Kontrolleure hier ]  [ Sie sind ausgestiegen ]  [ Ich bin ausgestiegen ]
```

- **Rückfrage an Eingecheckte** [S-01, S-06]: Ist in deinem Zug(teil) eine Meldung offen, fragt die Leiste am Halt: „Kontrolleure in deinem Zugteil? [Ja] [Nein] [Nur Security/Polizei] [weiß nicht]“. Höchstens 1 Frage pro Halt und 3 pro Fahrt.
- **Ausstiegsdialog für den Melder** [S-01]: „Kontrolleure noch im Zug? [noch drin] [auch raus] [weiß nicht]“.
- **Ruhige Darstellung im Check-in:** keine Vibration, kein Alarm für den eigenen oder gleich einfahrenden Zug [M-10].

---

## 5. Meldungslogik (das Herzstück)

### 5.1 Eine Meldung ist eine Beobachtung mit Anker

Eine Meldung beschreibt zuerst, **was wo wann gesehen wurde**. Der `fahrtKey` ist nur die beste Schätzung und kann später korrigiert werden [Z-01, Z-09, Z-10].

- **Anker:** Station, nächster Halt (= Richtung), Zeitpunkt der Beobachtung (nicht des Versands [T-01]), gesehene Linie und gesehenes Ziel.
- **Position gemeldeter Züge** wird ab dem jüngsten Anker berechnet. Der gemeldete Zug ist damit immer der am besten verortete, auch ohne Echtzeit.
- **Unklare Zuordnung:** Die Meldung wird auf mehrere Kandidaten verteilt (`kandidaten`), bis eine Beobachtung sie klärt.

| Typ | Bezug | Wirkung |
|---|---|---|
| `IM_ZUG` | Zug(teil) + Anker | Zugmeldung |
| `EINGESTIEGEN_IN` | vom Bahnsteig gesehen, Zug + Station | Zugmeldung |
| `AN_STATION` | Station (+ Ausgang) | Stationsmeldung, alle Linien |
| `GROSSKONTROLLE` | Station oder Bereich | Bereich, lange Dauer (5.5) |
| `BESTÄTIGT` / `KONTROLLIERT_GERADE` | Zug(teil) oder Station | stärkt die Meldung |
| `KEINE_GESEHEN` | eigener Zugteil | schwächt leicht |
| `AUSGESTIEGEN_IN` | Zug + Station | Zug höchstens 🟠, Station 🟠/🔴 (5.4) |
| `NICHT_MEHR_DA` / `FEHLALARM` | Zug oder Station | schwächt, löscht nie |
| `MELDER_RAUS` | Zug | „Melder nicht mehr an Bord“ |
| `ANWESEND` | Zug(teil), ohne Geräte-ID | Datenlage (5.2) |
| `DURCHFAHRT` | Fahrt, Station, Ist-Zeit | Crowd-Verspätung (6.4) |
| `KORREKTUR` / `ZURUECKGEZOGEN` | eigene frühere Meldung | umhängen / zurücknehmen |

### 5.2 Anzeige: zwei Achsen, Schätzung und Datenlage

| Symbol | Text | Bedeutung |
|---|---|---|
| 🔴 | gemeldet | frische Sichtung, bestätigt |
| 🟠 | wahrscheinlich / unsicher | ältere Sichtung, Melder ausgestiegen, „unklar“ oder nur eine ungeprüfte Meldung |
| ◌ (gestrichelt) | möglich | Vorhersage, z. B. Umstieg am Knoten. **Wird nie 🔴.** |
| ⚪ | keine Meldung · Nutzer an Bord | Beobachter da, niemand meldet etwas |
| ▒ (schraffiert) | keine Daten | niemand mit der App an Bord / auf dem Abschnitt |
| ? | Stand unbekannt | offline oder Server nicht erreichbar |

- **Datenlage:** Eingecheckte Geräte senden `ANWESEND`. Angezeigt wird nur grob „wenige / einige / viele Nutzer an Bord“ [T-02, S-10, K-10]. Zielkonflikt: Das zeigt auch dem Verkehrsunternehmen die Lücken [M-02, M-05]. Startentscheidung: pro Zug nur „Nutzer an Bord: ja/nein“, keine Zahlen.
- **Barrierefreiheit:** Status immer als Form + Text + Farbe, Screenreader-Labels, Wischen auch per Button [T-12].
- **Feste Texte:** „Keine Meldung heißt nicht frei.“ · „Kontrolleure in Zivil sind erst bei der Kontrolle erkennbar.“ · „Die App kann sich irren. Sicher ist nur ein gültiges Ticket.“
- **Grundrate:** kleines neutrales Etikett aus aggregierten Daten, z. B. „hier oft kontrolliert (Di 16–18 Uhr)“.

### 5.3 Wahrscheinlichkeit pro Zugteil, Abklingen pro Halt

Feste Timer (15/30 Minuten) waren auf langen Linien entweder zu kurz oder zu lang [S-05]. Kontrolleure steigen an Halten aus, also klingt die Meldung pro Halt ab [S-01].

- **Zustand je Zugteil:** `p` (Wahrscheinlichkeit, dass Kontrolleure an Bord sind), `letzteSichtung` (Station, Halt), `beobachterAnBord`.
- **Start bei `IM_ZUG`:** p = 0,85 mit Check-in, 0,6 ohne, jeweils mal Gerätegewicht (5.4).
- **An jedem Halt:** `p ← p · s(halt)`. Startwerte: 0,85 an normalen Halten, 0,7 an Knoten (Altona, Diebsteich, Dammtor, Hbf, Berliner Tor, Jungfernstieg, Ohlsdorf, Harburg, Bergedorf), 0,6 in Neugraben, 0 an der Endstation.
- **Beobachtungen** verändern p nach Bayes (Log-Odds): `BESTÄTIGT` stark nach oben, `KONTROLLIERT_GERADE` sehr stark, `KEINE_GESEHEN`, `NICHT_MEHR_DA`, `AUSGESTIEGEN_IN`, `FEHLALARM` nach unten. Startwerte stehen in [`loesungen_01`](szenarien/loesungen_01_veraltete_meldungen.md) (Abschnitt Q1).
- **Anzeige:** 🔴 nur bei p ≥ 0,6 **und** frischer Sichtung (Beobachter an Bord oder Sichtung am vorigen Halt). 🟠 ab 0,25. Keine Prozentzahlen auf dem Hauptscreen, weil sie Genauigkeit vortäuschen.
- **Ausbreitung an Halten:** Was an einem Halt „aussteigt“, verteilt sich als ◌ „möglich“ auf die Station (10 min) und auf Züge, die dort in den nächsten 10 Minuten abfahren, auch andere Linien und Gegenrichtung [S-04, S-08, K-05]. Höchstens 4 Hinweise pro Meldung, damit das Netz nicht flächig orange wird.
- **Fahrtende:** Die Meldung wird nicht gelöscht, sondern geht als 🟠 auf die Endstation (20 min) und als ◌ auf die Rückfahrt über [K-10].
- **Alle Handys rechnen gleich:** Die Parameter kommen versioniert mit der Fahrplandatei (6.1). Später werden sie aus aggregierten Daten kalibriert („nach wie vielen Halten steigen Kontrolleure aus?“).
- **Stationsmeldungen** verblassen nach 20 Minuten und verschwinden nach 45 Minuten, Großkontrollen erst nach mindestens 90 Minuten (5.5).

### 5.4 Missbrauch: Warnung leicht, Entwarnung schwer

Kontrolleure oder Trolle könnten falsche Entwarnungen schicken [M-01, M-04], Gruppen sich gegenseitig bestätigen [M-09], Emulatoren das Netz fluten [M-03].

1. **`AUSGESTIEGEN_IN` macht aus 🔴 höchstens 🟠.** ⚪ wird ein Zug nur durch Fahrtende, Abklingen oder Entwarnungen aus mindestens 2 unabhängigen etablierten Konten plus 10 Minuten ohne neue Warnung.
2. **Jede neue Warnung gewinnt** und hebt eine Entwarnung auf.
3. **Entwarnung löscht nie.** Der Verlauf bleibt sichtbar: „gemeldet 08:21 Blankenese · 1× als ausgestiegen gemeldet“.
4. **🔴 braucht mindestens 2 unabhängige Quellen** (andere Konten **und** anderer Ort oder ≥ 5 min später). Eine einzelne geprüfte Meldung ergibt 🟠. Ausnahme: frische Sichtung eines eingecheckten, etablierten Kontos.
5. **Konten statt freier Geräte-ID:** Schreiben nur mit App Attest (iOS) bzw. Play Integrity (Android). Neuinstallation auf demselben Gerät startet ohne Bonus (DeviceCheck / Device Recall).

| Stufe | Wann | Gewicht Warnung | Gewicht Entwarnung |
|---|---|---|---|
| W | Web, ohne Attest | 0,25 | 0 |
| S0 | neu (< 7 Tage oder < 3 fremd bestätigte Meldungen) | 0,5 | 0 |
| S1 | etabliert | 1,0 | 0,5 |
| S2 | bewährt (≥ 30 Tage, ≥ 10 fremd bestätigte Meldungen) | 1,5 (Deckel) | 0,75 (Deckel) |

6. **Serverseitige Plausibilität:** Meldungen werden gegen den Fahrplan geprüft (hält dieser Zug gerade hier?). Rate-Limit 1 Meldung pro 10 Minuten pro Konto. Antworten auf Rückfragen zählen nicht dagegen.
7. **Notbremse:** Bei ungewöhnlich vielen Meldungen (5-faches Mittel, beim Launch 3-fach) oder vielen gleichzeitigen Entwarnungen gelten zeitweise strengere Schwellen, und die App sagt das sichtbar („ungewöhnlich viele neue Melder, ungeprüft“).
8. **Großkontrolle:** Meldungen in einer laufenden Großlage sind Bestätigungen und werden nie als Spam verworfen [K-06].

### 5.5 Teams und Bereiche (nur auf dem Handy)

- **Team:** Neue Meldungen werden einem bestehenden Team zugeordnet, wenn Ort und Zeit plausibel zusammenpassen (gleicher Zug, Station, die der Zug passiert hat, oder Umstieg am Knoten). So erkennt die App, dass die Kontrolleure aus der S1 jetzt in der S3 sind, und nimmt die alte Meldung früher zurück [K-05, Z-15].
- **Bereich `PENDEL`:** mehrere Meldungen auf denselben 2–3 Stationen in kurzer Zeit → der Abschnitt wird markiert [K-03].
- **Bereich `GROSSLAGE`:** viele Meldungen an Knoten oder ausdrückliche Großkontroll-Meldung → Bereich mindestens 90 Minuten markiert [K-06].
- **Datenschutz:** Teams sind Bewegungsprofile einer kleinen, bestimmbaren Gruppe (rund 30 Mitarbeitende im Prüfdienst). Deshalb: nur auf dem Gerät berechnet, nie gespeichert oder übertragen, Spur höchstens 60 Minuten, Kette reißt nach 45 Minuten, keine Statistik pro Team. „Nur gesehen“-Meldungen bilden kein Team [K-11].

### 5.6 Flügelung, Vereinigung, Durchbindung, Wende

| Kopplung | Ort | Vererbung |
|---|---|---|
| `FLUEGELUNG` | S1 Ohlsdorf | mit Zugteil: dieser Teil 🔴, anderer kurz 🟠. Ohne Zugteil: beide 🟠 [Z-04, S-03] |
| `VEREINIGUNG` | S1 Ohlsdorf, Gegenrichtung | stärkerer Status beider Teile gilt für den ganzen Zug [Z-05] |
| `DURCHBINDUNG` | S5 Neugraben (zwei GTFS-Fahrten, ein Zug) | voll, bei unsicherer Kopplung 🟠 [Z-07] |
| `WENDE` | S7 Altona, alle Endstationen | 🟠, 🔴 nur wenn ein Eingecheckter an Bord bleibt [Z-08] |

Beim Check-in vor Ohlsdorf ergibt sich der Zugteil aus dem gewählten Ziel. Ohlsdorf gilt als Wechselfenster: Wer dort den Zugteil wechselt, wird nicht als ausgestiegen gewertet [T-04].

---

## 6. Wie die App Züge kennt

### 6.1 Datenpipeline (täglich prüfen)

```
hvv-GTFS (ZIP)
        │  Skript (Dart oder Python), GitHub Action, prüft täglich auf neue Version
        ▼
  1. nur S-Bahn-Linien behalten
  2. Stationen zusammenfassen (parent_station, DHID, Koordinaten, Ausgänge)
  3. Streifen je Linie und Richtung (häufigstes Haltemuster + Äste)
  4. Fahrten mit Haltezeiten und Verkehrstagen
  5. Kopplungen erkennen: S1-Flügelung/Vereinigung, S5-Durchbindung,
     S7-Wende in Altona, Wenden an Endstationen
  6. Netzgraph über alle Linien, Umstiegsknoten (inkl. U-Bahn aus demselben GTFS)
  7. erwartete Zuglänge (S3-Langzug werktags 5–19 Uhr)
  8. Parameter des Statusmodells (5.3)
        ▼
  sbahn-hh-<gültigAb>-<hash>.json  + manifest.json   (signiert)
        ▼
  GitHub Pages + weitere Spiegel
```

- **Korrektur zu Version 1:** Die S7 ist eine Schleife Aumühle → City-Tunnel → Altona → Verbindungsbahn → Aumühle. Im GTFS sind das zwei Fahrten, die in Altona **als Wende verbunden** werden müssen, sonst endet jede Meldung in Altona [Z-08].
- **Versionen:** Die App hält aktuelle und nächste Version und wählt je Betriebstag die passende. Sie rechnet **nie über das Gültigkeitsende hinaus**. Deckt keine Version den Tag ab, zeigt sie nur Stationsmeldungen [Z-14]. Der aktuelle Datensatz gilt nur bis 12.12.2026, der Fahrplanwechsel (S2 zurück nach Altona, S7 entfällt) muss also vorher eingespielt sein.
- **Linien nie hart codieren.** Alles wird aus dem GTFS erzeugt.
- **Namensnennung** „Hamburger Verkehrsverbund GmbH“ ist Pflicht (dl-de/by-2.0).

### 6.2 Zugposition und Uhrzeit

```dart
// Pseudocode
Position positionVon(Fahrt f, Instant jetzt, {Duration verspaetung = Duration.zero}) {
  final t = jetzt - verspaetung;
  for (var i = 0; i < f.halte.length - 1; i++) {
    final a = f.halte[i], b = f.halte[i + 1];
    if (t < a.abfahrt) return Position.amHalt(a);
    if (t < b.ankunft) return Position.zwischen(a, b, (t - a.abfahrt) / (b.ankunft - a.abfahrt));
  }
  return Position.beendet();
}
```

- **Verspätung** kommt (in dieser Reihenfolge) aus: jüngstem Anker einer Meldung (5.1), Crowd-Durchfahrten (6.4), DB Timetables (ab Phase 2).
- **Uhr:** Nie die Handy-Uhr. `jetzt` ist Serverzeit plus monotone Uhr des Geräts, GNSS-Zeit als Rückfall. Intern UTC, Betriebstag mit `Europe/Berlin` [T-05].
- **Reihenfolge:** Auf gemeinsamen Gleisen überholen sich S-Bahnen im Regelbetrieb nicht. Ein Zug kann nicht vor seinem Vordermann ankommen (mindestens ca. 3 min Abstand). Das hilft beim Zuordnen ohne Echtzeit [S-07, Z-01].
- **Darstellung:** hohler Punkt = nach Fahrplan, gefüllter Punkt = beobachtet vor X min.

### 6.3 Check-in: über das Ziel am Zug, nicht über die Fahrplanliste

Version 1 ließ den User „den nächsten Zug laut Fahrplan“ antippen. Bei Verspätung war das oft der falsche [Z-01, Z-02].

1. Der User tippt im Zug auf **„Ich bin eingestiegen“**.
2. Die Station kommt aus GPS (höchstens 150 m ungenau, höchstens 60 s alt), sonst aus Favorit oder letzter Station.
3. Die App zeigt **2–3 große Karten mit dem Ziel, das am Zug steht**, darunter Plan- und geschätzte Zeit. Liegen zwei Kandidaten unter 3 Minuten auseinander, fragt sie „Was steht am Zug?“.
4. **Schattenkandidaten** bleiben 30 Minuten erhalten. Neue Signale (GPS auf der Strecke, Verzweigung, Halte, Crowd-Daten) verschieben die Wahrscheinlichkeiten. Wird ein anderer Kandidat sehr wahrscheinlich, bucht die App um, sendet `KORREKTUR` und bietet „Rückgängig“ an.
5. **Station korrigieren** in der Fahrtleiste ist zugleich ein manueller Anker im Tunnel.

### 6.4 Crowd-Verspätung und Echtzeit

1. **Durchfahrten:** Eingecheckte Handys senden „Fahrt X, Station Y, Ist-Zeit“ mit einer zufälligen ID pro Check-in, ohne GPS-Rohdaten. Der Server löscht sie nach 3 Stunden.
2. **Prüfung:** Durchfahrten werden nur von eingecheckten Konten angenommen und als Median über mehrere Quellen genutzt. Ein einzelnes Konto kann keinen Zug verschieben [M-12].
3. **Ausfallverdacht:** Eine Fahrt ohne Beobachtung, deren Nachfolger schon gesehen wurde, ist verdächtig.
4. **DB Timetables API (ab Phase 2):** Ein kleiner Worker fragt die rund 70 S-Bahn-Stationen reihum ab (ca. 50 Aufrufe/min bei 60 erlaubten), ordnet die Züge über Linie, Planzeit, Ziel und eine Tabelle DHID ↔ EVA einem `fahrtKey` zu und verteilt Verspätungen und Ausfälle.
5. **gtfs.de GTFS-RT** parallel testen ⚠️. **Geofox GTI** nicht einplanen.

### 6.5 Check-in als Zustandsmaschine, Aussteigen erkennen

```
geplant → an_bord → evtl_raus → raus
```

- **„ausgestiegen“** erst nach zwei Signalen (Bewegungserkennung „gehend“ + GPS bleibt an der Station, oder Button) und kurzer Rückfrage. Haltezeiten und Ohlsdorf gelten als Wechselfenster [T-04].
- **Lebenszeichen:** Meldet sich das Handy nicht mehr oder ist es klar in einem anderen Verkehrsmittel, checkt die App automatisch aus. Nur aktive Check-ins bekommen Rückfragen und tragen Gewicht [Z-12].
- **Akku-Budget:** Bei wenig Akku checkt die App aktiv aus (`MELDER_RAUS` mit Grund) statt still zu verschwinden [T-13].
- **Kein Standort im Hintergrund.** Standort und Bewegung werden nur während eines Check-ins und **nur auf dem Gerät** ausgewertet.

| Signal | Im Tunnel? | Aussagekraft |
|---|---|---|
| Bewegungserkennung („im Fahrzeug“ → „gehend“) | ja | hoch, sparsam |
| GPS: User bleibt an Station, Zug fährt weiter | nein | hoch |
| Button „Ich bin ausgestiegen“ | ja | eindeutig |
| Fahrt hat Endstation erreicht | ja | automatisch |

---

## 7. Architektur

```
┌─────────────────────────┐        ┌────────────────────────────┐
│ App (Flutter)           │◀───────│ GitHub Pages + Spiegel     │
│ iOS · Android · Web/PWA │        │ sbahn-hh-*.json (signiert) │
│                         │        └────────────────────────────┘
│ - Fahrplan lokal        │
│ - Zugpositionen lokal   │  melden (REST, RPC)  ┌──────────────────────────────┐
│ - Status, Teams lokal   │ ───────────────────▶ │ Supabase                     │
│   aus bereinigten       │                      │ - Edge Function melden():    │
│   Ereignissen           │ ◀─────────────────── │   Attest, Rate-Limit,        │
│ - GPS nur lokal         │  bereinigte Ereignisse│   Plausibilität, Gewicht    │
│ - Offline-Queue         │  (Realtime je Linie) │ - Rohtabelle: per RLS nicht  │
└─────────────────────────┘                      │   lesbar                     │
                                                 │ - Löschjob                   │
                ┌──────────────────────┐         └──────────────────────────────┘
                │ Echtzeit-Worker       │ ──▶ Verspätungen/Ausfälle (ab Phase 2)
                │ (DB Timetables)       │
                └──────────────────────┘
```

**Wer rechnet was:** Der Server nimmt Meldungen an, prüft sie und verteilt sie **bereinigt**: ohne Konto- oder Geräte-ID, mit Gewichtsstufe statt Konto, mit auf die Minute gerundeter Zeit und kleiner zufälliger Verzögerung (wenige Sekunden). Status, Teams und Positionen berechnet jedes Handy deterministisch aus diesen bereinigten Ereignissen. So ist niemand aus dem Stream identifizierbar [M-06], und das Backend bleibt klein.

> Zielkonflikt: Die Lösung zu M-05 schlug vor, nur den fertig berechneten Zustand auszuliefern und 10–40 s zu verzögern. Das schützt besser vor Mitlesen, macht aber Teams (5.5) und das Statusmodell (5.3) auf dem Handy unmöglich und kostet Aktualität. Startentscheidung: bereinigte Einzelereignisse, kurze Verzögerung. Siehe offene Fragen.

**Betrieb:**
- **Offline-Queue** auf dem Gerät, übersteht App-Neustarts. Jede Meldung trägt eine eindeutige `client_event_id`, damit nichts doppelt ankommt [T-01].
- **Fail loud:** Ist der Server nicht erreichbar oder voll, zeigt die App „? Stand unbekannt“ statt ⚪ [T-06]. Eine statische `status.json` auf den Spiegeln erlaubt einen Notbetrieb.
- **Cache:** Alles aus dem Cache trägt sichtbar „Stand: HH:MM“, Meldungen vom Vortag werden nie angezeigt [T-07].
- **Kosten:** Das Free-Tier reicht für Tests. Für den Launch den bezahlten Tarif mit Ausgabenlimit einplanen ⚠️ aktuelle Limits und Preise prüfen.

### Datenmodell

```sql
-- Konto: anonym, pro attestierter Installation, ohne Orte und ohne Meldungshistorie
create table konten (
  id            text primary key,       -- HMAC(server_secret, attest_key_id)
  stufe         text not null,          -- W, S0, S1, S2
  erstellt_woche date not null,
  bestaetigt_n  int not null default 0
);

-- Rohmeldungen: per RLS für Clients NICHT lesbar
create table meldungen (
  id              uuid primary key default gen_random_uuid(),
  client_event_id uuid unique not null,  -- gegen Doppelte aus der Offline-Queue
  typ             text not null,         -- siehe 5.1
  fahrt_key       text,                  -- beste Schätzung, korrigierbar
  kandidaten      jsonb,                 -- [{fahrt_key, p}] bei unklarer Zuordnung
  station_id      text,                  -- DHID, Ort der Beobachtung
  naechste_station_id text,              -- Richtung
  ausgang_id      text,
  linie_gesehen   text,
  ziel_gesehen    text,
  einheit         smallint,              -- 1 = vorne … 3; null = unbekannt
  seit_station_id text,                  -- "waren schon vorher da"
  aktiv           boolean,               -- true kontrollieren, false nur gesehen
  sicher          boolean not null default true,
  gross           boolean not null default false,
  quelle          text,                  -- checkin | bahnsteig | rueckfrage
  bezug_id        uuid,                  -- Bestätigung/Korrektur bezieht sich auf …
  datensatz       text not null,         -- Fahrplanversion auf dem Gerät
  beobachtet_um   timestamptz not null,  -- Serverzeit zum Zeitpunkt des Tippens
  empfangen_um    timestamptz not null default now(),
  konto_tag       text,                  -- Tagespseudonym, nur bis Ereignisende
  gewicht         real not null
);
```

**Löschung:** Rohmeldungen werden **2 Stunden nach Ereignisende** gelöscht (statt 24 h), `konto_tag` schon bei Ereignisende. Durchfahrten nach 3 h, Anwesenheit nach wenigen Minuten (nur Zähler). Für Statistik bleiben nur Aggregate pro Station, Wochentag und Stunde. Tests für RLS und Löschjob laufen in der CI, ein Alarm meldet, wenn der Löschjob ausfällt [M-07].

---

## 8. Recht, Sicherheit, Store und Name

Keine Rechtsberatung.

- **In Deutschland gibt es kein Verbot** solcher Apps und kein bekanntes Urteil. Juristen diskutieren Unterlassungsansprüche und Beihilfe, die Lage ist ungeklärt. **Vor dem Launch: Anwalt für IT-Recht.**
- **Name und Ton neutral.** „Schwarzfahr“ bleibt Repo-Name, die App heißt anders und fordert nicht zum Fahren ohne Ticket auf. In der App steht: „Sicher ist nur ein gültiges Ticket.“
- **Keine Logos, keine offiziellen Netzpläne**, nicht „S-Bahn Hamburg“ oder „hvv“ im Namen.
- **Keine Fotos, kein Freitext, keine Personenbeschreibungen.** Teams nie speichern (5.5).
- **Sicherheitsregeln** [M-10, M-11]:
  - kein Push und kein Alarm für den eigenen oder gleich einfahrenden Zug
  - keine Handlungs- oder Ausweichempfehlungen („steig aus“, „fahr über Dammtor“)
  - Push später nur als Linien-Abo ohne gespeicherte Tokens, ohne Freitext
  - ein späterer Telegram-Kanal nur mit Liniensummen und ohne Freitext-Eingang
- **Datenschutz bei Dritten:** Auch Hoster, Push-Dienst und Telegram speichern Metadaten. Hoster-Logs so kurz wie möglich halten, das in der Datenschutzerklärung offen sagen, auch dass Mitlesende die öffentliche Anzeige archivieren können [M-05, M-07].
- **Impressum:** Name und Anschrift sind Pflicht. Ein Verein (wie FreiFahren) schützt die Privatperson.
- **Store-Entfernung einplanen:** Web-App/PWA und APK als vorab angekündigter Ausweichweg. Die Fahrplandatei kommt von mehreren Spiegeln, damit auch alte Installationen Updates bekommen [T-13]. Die Store-Beschreibung muss zeigen, was die App tatsächlich tut.
- **Barrierefreiheit** (BFSG) von Anfang an mitdenken (5.2).

---

## 9. Kaltstart

1. **Nutzen ohne Meldungen:** Abfahrtstafel und Zugpositionen. Ab dem öffentlichen Start mit Echtzeit, sonst ist sie schlechter als die hvv-App.
2. **Klein und dicht starten:** erst die S1, dann alle S-Bahn-Linien.
3. **Gezielter Launch:** r/hamburg und Uni-Gruppen.
4. **Keine Paywall vor Meldungen.** Spenden oder Unterstützer-Abo ohne Funktionssperre.
5. **Ehrlich zu dünner Datenlage:** Schraffur und Hinweis „auf dieser Strecke heute wenig Nutzer“ statt leerer, scheinbar ruhiger Karte.
6. **Start-Notbremse** mit strengeren Schwellen in den ersten Wochen, weil Sabotage beim Launch am wahrscheinlichsten ist [M-08].

---

## 10. Fahrplan der Umsetzung

### Phase 0 – Daten und Prototyp (1–2 Wochen)
- [ ] GTFS-Skript: S-Bahn filtern, Streifen, Fahrten, neuer `fahrtKey`
- [ ] Kopplungen erkennen (Flügelung, Vereinigung, S5-Durchbindung, S7-Wende), Netzgraph
- [ ] Versionen + Manifest, tägliche Prüfung, Signatur
- [ ] Tests gegen die Recherche (z. B. ~12 Züge je Richtung S1 in der HVZ)
- [ ] Flutter-Prototyp: Streifen, Züge nach Fahrplan, Karten wischen, zentrale Uhr

### Phase 1 – MVP für Testgruppe auf der S1 (3–5 Wochen)
- [ ] Abfahrtstafel, Zugkarte, Linienübersicht, Verbindungsleiste, alle Anzeige-Stufen aus 5.2
- [ ] Melden mit Anker, Nachtrag-Chips, Rückgängig, Offline-Queue
- [ ] Check-in über Ziel, Schattenkandidaten, Station korrigieren
- [ ] Statusmodell pro Halt (5.3), Entwarnung schwer (5.4)
- [ ] Edge Function `melden()`, Attest, Konten-Stufen, RLS, Löschjob mit CI-Tests
- [ ] Web-Version + TestFlight / Android-Test
- [ ] Messen: Anteil korrigierter Check-ins in der HVZ

### Phase 2 – Genauigkeit und öffentlicher Start (3–5 Wochen)
- [ ] **Echtzeit-Worker mit DB Timetables API** (vorgezogen aus Phase 3)
- [ ] Rückfragen an Eingecheckte, Ausstiegsdialog, Check-in-Zustandsmaschine
- [ ] Crowd-Durchfahrten mit Median-Prüfung
- [ ] Teams und Bereiche (5.5), Großkontrolle
- [ ] Datenlage-Anzeige, Grundrate, Notbremse
- [ ] Anwaltliche Prüfung, dann öffentlicher Start

### Phase 3 – Wachstum
- [ ] Push als Linien-Abo (ohne Token-Speicherung, nicht für den eigenen Zug)
- [ ] Kalibrierung der Parameter aus aggregierten Daten
- [ ] U-Bahn Hamburg, dann andere Städte über bundesweite GTFS-Daten

### Kosten zum Start
| Posten | Kosten |
|---|---|
| Supabase Free Tier (Test) / bezahlter Tarif (Launch) | 0 € / ca. 25 $ pro Monat ⚠️ prüfen |
| GitHub Pages + Spiegel für die Fahrplandatei | 0 € |
| Apple Developer Program | 99 €/Jahr |
| Google Play | 25 € einmalig |
| Anwaltliche Erstberatung | grob 150–400 € (Schätzung) |

---

## 11. Offene Fragen

1. **Name der App** (neutral, ohne Marken).
2. **Allein oder als Verein?** Folgen für Impressum und Haftung.
3. **Wie viel Datenlage zeigen?** Mehr Abdeckungsanzeige macht ⚪ ehrlicher, verrät aber auch die Lücken an das Verkehrsunternehmen (5.2).
4. **Wie viel Verzögerung im Stream?** Bereinigte Einzelereignisse mit kurzer Verzögerung (Startentscheidung) oder nur fertiger Zustand mit 10–40 s Verzögerung (mehr Schutz, weniger Funktion) (7).
5. **Zugteil in Ohlsdorf:** Fährt der vordere Teil wirklich zum Airport? Vor Ort prüfen und als Daten pflegen, nicht als Konstante.
6. **Parameter** (Abklingen pro Halt, Gewichte, Schwellen) sind Startwerte und müssen mit echten Daten nachjustiert werden.
7. **Supabase-Limits und Preise** mit der aktuellen Preisliste abgleichen.
