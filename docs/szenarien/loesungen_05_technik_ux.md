# Lösungen T: Technik, Nutzerdichte und Bedienung

Bezug: [`05_technik_ux.md`](05_technik_ux.md), [`../KONZEPT.md`](../KONZEPT.md). Stand: 3. November 2026.

**Faktenkorrekturen an den Szenarien** (für die Lösungen ohne Belang, aber für Tests und Texte wichtig):
- T-01: Von Hbf über Jungfernstieg nach Landungsbrücken fährt die S1 **Richtung Wedel**, nicht Richtung Poppenbüttel.
- T-03: Die S3 hält nicht an der Sternschanze. Über die Verbindungsbahn fahren S2, S5 und S7. Gemeint ist z. B. die S3 Richtung Pinneberg ab Reeperbahn mit Favorit Altona.
- T-04: Zwischen zwei Einheiten der BR 474 kann man nicht durchgehen. Wer in Ohlsdorf in den Airport-Teil wechselt, steigt kurz auf den Bahnsteig aus. Das macht die Fehlerkennung noch wahrscheinlicher.
- T-05: Holstenstraße bedienen S5 und S7 (nachts S2), nicht die S3.
- T-07: Die App zeigt nie Grün. „Alles grün“ meint ein ⚪ aus dem Cache.
- T-08: **S21 und S31 gibt es seit 10.12.2023 nicht mehr** (heute S2 und S5). An Dammtor halten S2 (HVZ), S5 und S7, ab Dezember 2026 voraussichtlich S2 und S5.
- T-10: Das Konzept hat keine Konten, nur anonyme Geräte-IDs.
- T-11: Laut Konzept werden keine GPS-Koordinaten gespeichert. Das Profilrisiko besteht trotzdem, weil Geräte-ID, Station und Zeit zusammen gespeichert werden.
- T-13: Der Fahrplanwechsel am So 13.12.2026 passt. Laut Konzept 6.1 lädt die App den Fahrplan vom CDN und nicht über ein App-Update. Ein fehlendes Store-Update stoppt den Datensatz also nicht. Das Risiko liegt bei Formatwechseln und bei abgelaufenen Datensätzen auf Geräten, die nie neu laden.

---

### T-01 Tunnel-Meldung kommt zu spät (City-S-Bahn)
- **Lösung:**
  1. **Persistente Offline-Queue** (SQLite/Drift, übersteht App-Neustart). Jede Meldung bekommt beim Tippen:
     - `client_event_id` (UUID v4) für Idempotenz. Der Server hat `unique(client_event_id)`, darum erzeugen Wiederholungen keine Dubletten.
     - `beobachtet_um`: der Moment des Tippens in **Serverzeit** (siehe T-05), nicht der Moment des Versands.
     - `fahrt_key` aus dem Check-in, dazu `station_id` = letzte passierte Station und `naechste_station_id`, wenn der Zug zwischen zwei Stationen ist. Beides wird beim Tippen eingefroren, Grundlage sind die Fahrplanposition und die Crowd-Verspätung. Bei der Ankunft wird nichts neu berechnet.
     - `zeitquelle` ∈ {`server`, `gnss`, `geraet`} und `datensatz_version`.
  2. **Versand:** sofort versuchen. Danach erneut bei jedem Wechsel der Verbindung (`connectivity_plus`), bei jedem Halt laut Fahrplan und mit Backoff (2 s, 5 s, 15 s, 30 s, Jitter). Der Server prüft `beobachtet_um ≤ now() + 30 s` und `≥ now() − 60 min`. Ältere Meldungen gehen nur noch in die Statistik.
  3. **Statuslogik nutzt `beobachtet_um`**, nicht `empfangen_um`. Alle Timer aus 5.2 (15 min, 30 min) laufen ab Beobachtung. Eine nachgereichte Meldung erscheint deshalb mit dem richtigen Alter am richtigen Zug.
  4. **Anzeige:** „Kontrolle gemeldet 08:12 zwischen Hbf und Jungfernstieg · eingetroffen 08:16“. Den zweiten Teil gibt es nur, wenn die Differenz größer als 60 s ist.
  5. **Melder sieht den Queue-Zustand** in der Fahrtleiste (4.5): „⏳ Meldung wartet auf Netz (seit 08:12)“, danach „✓ zugestellt 08:16“, „✗ nicht zugestellt – erneut senden“. Nichts wird still verworfen.
  6. **Tunnel ehrlich machen:** Das Datenskript markiert Abschnitte mit `funkloch: true`, manuell gepflegt, z. B. den City-Tunnel Hbf–Altona und den Harburger Tunnel. Fährt ein Zug laut Position in so einem Abschnitt, zeigt die Karte „📶 Tunnelabschnitt – Meldungen aus diesem Zug können verzögert kommen“. Hat der Zug Beobachter (T-02), deren letzter Heartbeat älter als 3 min ist, zeigt sie: „2 Nutzer an Bord, gerade ohne Netz“.
  7. Verworfen: Weiterleitung von Handy zu Handy per Bluetooth (zu komplex, verrät, wer die App nutzt).
- **Status:** 🟡 teilweise. Zeitstempel und Station sind ✅ korrekt, die Verzögerung selbst lässt sich ohne Netz nicht vermeiden.
- **Restrisiko:** B sieht die Meldung erst, wenn A wieder Netz hat. Die App kennzeichnet den Zug als „Tunnel / Augen offline“ statt als neutrales ⚪, damit B die Lücke erkennt.

### T-02 Außenast früh morgens: keine Meldung heißt „niemand da“, nicht „sicher“
- **Lösung:**
  1. **Abdeckung messen (Präsenz):** Geräte mit bestätigtem Check-in (T-04) senden alle 2 min und bei jedem Halt einen Heartbeat `{fahrt_key, zugteil, sitzung_id}`. `sitzung_id` ist eine Zufallszahl pro Check-in. Der Heartbeat landet in einer `unlogged`-Tabelle `praesenz` mit TTL 8 min. Ein `pg_cron`-Job verdichtet einmal pro Minute in `abdeckung(bezug, anzahl_bucket, stand)` und löscht dabei alles, was älter ist. Der Bezug ist eine Fahrt oder ein Streckenabschnitt. Es werden keine Geräte-IDs gespeichert, auch keine Präsenz-Historie.
  2. **Stationsbeobachter:** Wer die Abfahrtstafel einer Station im Vordergrund hat und per GPS höchstens 300 m entfernt ist, zählt für 10 min als Beobachter dieser Station.
  3. **Buckets statt genauer Zahl:** 0 / „wenige“ (1–2) / „einige“ (3–9) / „viele“ (≥ 10). Bei 1 Nutzer wird so niemand bloßgestellt.
  4. **Neuer Anzeigezustand:**
     - ⚪ „Keine Meldung · wenige Nutzer an Bord“, nur wenn `anzahl ≥ 1`.
     - ◌ (gestrichelter Kreis) **„Keine Daten – niemand mit der App in diesem Zug“** bei `anzahl = 0`.
     - Auf dem Streifen (4.3) werden Abschnitte ohne Beobachter in den letzten 30 min schraffiert dargestellt, mit dem Text „S5 Neugraben–Stade: gerade niemand aktiv“.
  5. **Zugteil:** Ist die Abdeckung bekannt, gibt es den Zusatz „Nutzer sehen nur ihren Wagen“. Der Check-in fragt optional „vorne / Mitte / hinten“ (bei langen Zügen relevant: S3 als Langzug, S1 vor Ohlsdorf).
  6. **Grundrate statt Leere:** Aus den aggregierten Statistiken (Konzept 7) entsteht pro Abschnitt, Wochentag und Stundenband: „Hier wurde in den letzten 8 Wochen an 5 von 40 Werktagen zwischen 5 und 7 Uhr eine Kontrolle gemeldet“. Ab weniger als 20 Meldungen insgesamt steht dort „zu wenig Daten“. Hinweis aus der Recherche: Seit 2026 kontrollieren vhh-Prüfteams gezielt im Umland (Pinneberg, Stade), auch in der S-Bahn.
  7. Kaltstart (Konzept 9): Check-in lohnt sich auch ohne Meldungen (Aussteige-Erinnerung, Verspätung). Jeder Check-in erhöht die Abdeckung.
- **Status:** 🟡 teilweise. Die Anzeige wird ehrlich ✅, die Datenlücke auf Außenästen bleibt.
- **Restrisiko:** Auf Stade, Wedel und Aumühle wird häufig ◌ stehen. Die App sagt das offen, statt Sicherheit vorzutäuschen. Das kann Nutzer kosten, ist aber ehrlicher als ein falsches ⚪.

### T-03 Push kommt zu spät (Doze-Modus, iOS-Drosselung)
- **Lösung:**
  1. **Push-Inhalt auf den Zug beziehen, Alter mitschicken:** Die Nutzlast enthält `fahrt_key, typ, station_id, beobachtet_um`. Der Text entsteht **auf dem Gerät beim Eintreffen**:
     - Android: Data-Message, die App baut die Notification lokal.
     - iOS: `mutable-content: 1` plus Notification Service Extension. Sie läuft auch, wenn die App beendet wurde.
     - Beide rechnen aus dem lokalen GTFS die aktuelle Position aus: „S3 → Pinneberg (ab Hbf 16:58): Kontrolle gemeldet **vor 9 min** an Reeperbahn. Zug ist jetzt etwa in Altona.“
  2. **Verfallszeit:** FCM `ttl = 600 s`, APNs `apns-expiration = jetzt + 600`. Ist die Meldung beim Eintreffen älter als 8 min, zeigt die App sie nur noch still als „veraltet“ oder gar nicht (Einstellung).
  3. **Priorität korrekt nutzen:**
     - FCM `android.priority = high` nur für Meldungen, die tatsächlich angezeigt werden. Sonst stuft Android die App herunter.
     - iOS `apns-priority 10`, `interruption-level: time-sensitive` (Capability „Time Sensitive Notifications“).
     - In den Einstellungen gibt es einen Hinweis mit Deep-Link auf die Akku-Optimierung (`ACTION_IGNORE_BATTERY_OPTIMIZATION_SETTINGS`). Nicht verwendet wird die Direktanfrage `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`, sie ist Play-Policy-kritisch.
  4. **Push-Grenzen:** nur Favoriten-Strecken in selbst gewählten Zeitfenstern (z. B. Mo–Fr 07–09), höchstens 4 Pushs pro Stunde, keine Entwarnungs-Pushs.
  5. **Zuverlässigere Alternative ohne Server-Push:** lokale, vom Gerät geplante Erinnerung („Dein Zug 08:12 ab Altona – jetzt Status prüfen“, `zonedSchedule` / `UNCalendarNotificationTrigger`). Beim Antippen lädt die App den Live-Stand. Lokale Alarme werden vom System deutlich verlässlicher zugestellt als Fremd-Pushs.
  6. **Datenschutz:** FCM-Topics pro Station und Linie (`st_<id>`, `ln_<linie>_<richtung>`). Der eigene Server speichert **keine Push-Tokens**. Eine Edge Function (Datenbank-Webhook auf `meldungen`) sendet an das Topic. Push bleibt Opt-in (Konzept 10, Phase 3).
  7. Wording in den Einstellungen: „Mitteilungen können vom Handy verzögert werden. Verlass dich vor dem Einsteigen auf den Live-Stand in der App.“
- **Status:** 🟡 teilweise. Die falsche Station und das fehlende Alter sind ✅ gelöst, die OS-Verzögerung bleibt.
- **Restrisiko:** Doze, App-Standby-Buckets und der iOS-Fokusmodus können Pushs weiterhin verzögern oder unterdrücken. Ein verspäteter Push sagt aber, wie alt er ist und wo der Zug *jetzt* ist.

### T-04 Bewegungserkennung meldet „ausgestiegen“, obwohl der Nutzer noch im Zug ist
- **Lösung:** eine Zustandsmaschine für den Check-in statt eines einzelnen Signals.
  1. **Zustände:**
     - `geplant`: nach „Ich steige ein“.
     - `an_bord`: erst nach einem Beleg, nämlich `in_vehicle` nach der planmäßigen bzw. crowd-korrigierten Abfahrt, oder GPS-Geschwindigkeit über 15 km/h entlang des Shapes, oder ein Tap auf „Ich bin drin“.
     - `evtl_raus`, danach `raus`.
  2. **Meldungen aus `geplant`:** Ein `IM_ZUG` aus diesem Zustand fragt zuerst „Bist du schon im Zug? [Im Zug] [Am Bahnsteig]“. So landet H's Beobachtung als `AN_STATION` und hängt nicht an einer fremden Fahrt. Wer 3 min nach der Abfahrt noch nicht `an_bord` ist, wird mit sichtbarem Hinweis ausgecheckt („Check-in beendet: Du bist nicht losgefahren“).
  3. **`an_bord` → `evtl_raus`** nur bei mindestens 2 Signalen:
     - `walking` oder `still` für mindestens 60 s,
     - der Zug ist laut Fahrplan abgefahren,
     - GPS (falls vorhanden) mehr als 300 m von der berechneten Zugposition entfernt oder ortsfest an der Station.
  4. **`evtl_raus` → `raus`:** Die App schickt eine lokale Mitteilung „Bist du ausgestiegen? [Ja] [Nein, noch drin]“. Ohne Antwort und ohne erneutes `in_vehicle` innerhalb von 2 min sendet sie `MELDER_RAUS`. Bis dahin bleibt der Zugstatus unverändert.
  5. **Haltezeit gilt als Wechselfenster:** Gehen während eines Halts, auf das innerhalb von 90 s nach der Abfahrt wieder `in_vehicle` folgt, ist ein Wagenwechsel und kein Ausstieg. In **Ohlsdorf** (Flügelung, BR 474 nicht durchgängig) fragt die App bei Gehen aktiv: „Zugteil gewechselt? [Airport (vorne)] [Poppenbüttel (hinten)] [Ausgestiegen]“. Ohne Antwort ordnet die App den Zugteil nach der Abfahrt per GPS über den Ast zu (Airport oder Poppenbüttel, binnen 1–2 min eindeutig). Bis dahin erben beide Teile die Meldung als 🟠 (Konzept 5.4).
  6. **Latenz einplanen:** Activity Recognition (Transition API) und CoreMotion melden erst nach 30 s bis 2 min. Darum gelten keine harten Grenzen unter 60 s.
- **Status:** 🟡 teilweise.
- **Restrisiko:** Im Tunnel ohne GPS und ohne Antwort des Nutzers bleibt die Heuristik fehleranfällig. Dann wird der Zug eher zu früh 🟠, er wird nie fälschlich 🔴. Zugteil-Wechsel ohne Antwort sind bis zur Ast-Trennung unklar, die Anzeige sagt dann „Zugteil unklar“.

### T-05 Falsche Handy-Uhrzeit verschiebt alle Züge
- **Lösung:**
  1. **Eine zentrale `Uhr`-Klasse**, die überall statt `DateTime.now()` gilt (Lint-Regel, die `DateTime.now` außerhalb dieser Klasse verbietet).
  2. **Serverzeit-Sync:** Die RPC `zeit()` liefert `now()` (Postgres-Funktion, kein Edge-Function-Aufruf). Die App misst t0 und t1 und berechnet `offset = server − (t0 + t1)/2`. Gültig sind nur Messungen mit RTT unter 2 s, die App nimmt den Median aus 3 Messungen. Synchronisiert wird bei Start, bei Resume und alle 15 min.
  3. **Monotone Basis:** Die App speichert `(serverzeit, elapsedRealtime)` als Anker, über einen Plattform-Kanal (`SystemClock.elapsedRealtime` bzw. `systemUptime` / `CLOCK_MONOTONIC`). `jetzt = anker.server + (elapsed − anker.elapsed)`. Verstellt der Nutzer die Uhr, ändert das nichts. Nach einem Neustart (`elapsed` < Anker) ist der Anker ungültig.
  4. **Ohne Netz beim Start:** Bei einem GPS-Fix liefert Android `Location.getTime()` die GNSS-UTC-Zeit, `zeitquelle = 'gnss'`. Sonst gilt die Gerätezeit mit `zeitquelle = 'geraet'` und Banner „Uhrzeit nicht geprüft“.
  5. **Abweichung offen sagen:** Weicht die Geräteuhr mehr als 60 s ab: „Deine Handy-Uhr geht 6 min nach. Die App rechnet mit der richtigen Zeit.“
  6. **Zeitzone und Zeitumstellung:** intern alles in UTC. Für GTFS gilt die Service-Tag-Semantik („Mittag minus 12 h“, Zeiten über 24:00) mit `Europe/Berlin` aus dem Paket `timezone`. Tests decken die Umstellungsnächte ab (z. B. 25.10.2026 und 28.03.2027).
  7. **Server-Plausibilität:** Für `IM_ZUG` berechnet die RPC `melden()` aus dem Datensatz (T-13), wann die gemeldete Fahrt an `station_id` sein sollte. Weicht das mehr als den halben Takt plus die bekannte Verspätung von `beobachtet_um` ab, reagiert der Server: Hat genau eine andere Fahrt derselben Linie und Richtung eine passende Zeit, wird die Meldung auf diese korrigiert. Sonst bekommt sie Gewicht 0,3 (🟠). Die App erfährt die Korrektur und zeigt sie an.
- **Status:** ✅ gelöst.
- **Restrisiko:** Gerät erstmals offline, ohne GPS und mit falscher Uhr: Die Meldung geht mit `zeitquelle = 'geraet'` raus, der Server ordnet sie über `empfangen_um` neu ein und gewichtet sie geringer. Die Anzeige sagt das.

### T-06 Supabase-Ausfall oder Free-Tier-Limit erreicht
- **Lösung:**
  1. **Fail-loud: Verbindungsleiste oben auf jedem Screen**, abgeleitet aus dem Status des Realtime-Kanals und dem letzten Heartbeat-Reply (Phoenix, alle 30 s):
     - „● Live“ (neutrales Symbol, kein Grün): Kanal `SUBSCRIBED`, letzter Reply unter 60 s.
     - „◐ Stand vor 3 min“
     - „✕ Offline – Stand 21:58. Neue Meldungen fehlen.“
     - **Ab 5 min ohne Live-Stand** werden ⚪ und ◌ durch **„? Stand unbekannt“** ersetzt. 🔴 und 🟠 bleiben mit ihrem Alter stehen.
  2. **Schreiben unabhängig von Realtime:** Meldungen gehen per PostgREST-RPC `melden()` (Security-Definer-Funktion: Rate-Limit, Validierung, `realtime.send()` als Broadcast aus der DB). Volle Realtime-Verbindungen blockieren so keine Meldungen. Fehler landen sichtbar in der Queue (T-01) mit Grund: „Server überlastet – wird erneut gesendet“.
  3. **Nach jedem Reconnect nachladen:** Realtime spielt Verpasstes nicht nach. Darum fragt die App `meldungen where empfangen_um > letzter_stand − 2 min` per REST ab.
  4. **Fallback-Polling:** Wird Realtime abgelehnt, fragt die App im Vordergrund alle 45 s per REST mit `id > letzte_id` ab. Antworten sind meist leer, also wenige hundert Byte.
  5. **Kosten senken:** ein Broadcast-Kanal pro Linie statt `postgres_changes`. Abonniert werden nur Linien in der Nähe bzw. Favoriten. Die Verbindung wird 2 min nach dem Wechsel in den Hintergrund getrennt. Status-Deltas werden alle 5 s gebündelt.
  6. **Unabhängige Notbetriebs-Seite:** `status.json` auf GitHub Pages (gleicher Ort wie der Datensatz) mit `{notbetrieb, text, seit}`. Eine GitHub Action prüft alle 5–10 min die Erreichbarkeit, setzen kann man den Wert auch manuell. Die App zeigt den Text als Banner.
  7. **Budget (Supabase-Preisliste 2025, vor Launch prüfen):**
     - Free-Tier: 200 gleichzeitige Realtime-Verbindungen, 2 Mio. Nachrichten pro Monat, 5 GB Egress, 500 MB DB. Pausiert nach 7 Tagen Inaktivität.
     - Grobe Schätzung: Das reicht für etwa 1.000–2.000 Nutzer pro Tag. Ein viraler Post sprengt das.
     - Darum: Nutzungsalarm bei 70 %, Upgrade auf **Pro (25 $/Monat, 500 Verbindungen, 5 Mio. Nachrichten)** mit Spend-Cap fertig vorbereiten und in der Kostentabelle (Konzept 10) einplanen.
     - Zeitsync und Abdeckung (T-02, T-05) nicht minütlich pollen, sonst ist der Egress schnell aufgebraucht.
  8. Telegram-Kanal (Konzept 9.3) als Ausweichweg im Banner verlinken.
- **Status:** ✅ gelöst (die stille Fehlanzeige verschwindet, Limits sind eingeplant).
- **Restrisiko:** Ein echter Ausfall bleibt möglich. Dann sieht jeder „? Stand unbekannt“ statt eines falschen ⚪, und unzugestellte Meldungen bleiben in der Queue sichtbar.

### T-07 Gecachte Meldungen von gestern
- **Lösung:**
  1. **Nur absolute Zeitstempel:** Jede Meldung (auch `AN_STATION`) trägt `beobachtet_um timestamptz`. Kein Feld speichert nur eine Uhrzeit. Der Cache speichert die Meldungen und dazu `cache_stand` in Serverzeit plus den `elapsedRealtime`-Anker (T-05).
  2. **Ablauf beim Laden anwenden:** Beim Start filtert die App den Cache mit den Regeln aus 5.2 gegen `Uhr.jetzt()`. Was abgelaufen ist (Station nach 45 min, Fahrt nach Fahrtende), wird nicht angezeigt. Eine Stationsmeldung von gestern 18:30 kann morgens deshalb nie erscheinen.
  3. **Datenfrische auf dem Screen:** Ist der Cache älter als 5 min, gilt die Verbindungsleiste aus T-06 („✕ Offline – Stand gestern 18:30“), und ⚪/◌ werden zu „? Stand unbekannt“.
  4. **Uhr unbekannt** (Neustart, offline, kein GPS): Der Cache gilt als veraltet und wird nur mit Datum angezeigt („Stand: 02.11., 18:30“), ohne Statusfarben.
- **Status:** ✅ gelöst.
- **Restrisiko:** Offline sieht J keine aktuellen Meldungen. Die App sagt das deutlich, statt alte Daten als aktuell auszugeben.

### T-08 Meldung aus Versehen in der Hosentasche
- **Lösung:**
  1. **Kein Ein-Tap-Melden:** Der Button öffnet ein Sheet (4.4). Die Bestätigung ist ein **Halten-Button (600 ms, mit Fortschrittsring)** an anderer Stelle als der Öffnen-Button. Beim Tragen in der Tasche passieren zwei Taps an verschiedenen Stellen plus Halten praktisch nie.
  2. **Näherungssensor:** Solange er „nah“ meldet (Tasche, Ohr), nimmt das Melde-Sheet keine Eingaben an (`proximity_sensor`, iOS `isProximityMonitoringEnabled`).
  3. **Rückgängig 10 s:** Die Meldung wird **sofort** gesendet, Verzögerung kostet Zeit. Dazu erscheint 10 s lang die Snackbar „Gemeldet: Bahnsteig Dammtor · [Rückgängig]“. Rückgängig sendet `ZURUECKGEZOGEN` mit derselben `client_event_id`, der Server löscht die Meldung und broadcastet das Entfernen. Danach kann der Melder 15 min lang über „Meine letzte Meldung“ widerrufen.
  4. **Gewicht:** Eine einzelne, unbestätigte Meldung ohne Check-in (Stationsmeldung per GPS) ist 🟠 „1 Meldung, unbestätigt“. Erst eine zweite unabhängige Meldung oder ein eingecheckter Melder macht sie 🔴 (Konzept 11.4). Unbestätigte Stationsmeldungen verblassen nach 20 min (5.2).
  5. Wer zum ersten Mal meldet, sieht einmalig einen Hinweis: „Melde nur, was du selbst siehst.“
- **Status:** ✅ gelöst.
- **Restrisiko:** Eine bewusste Fehlbedienung lässt sich so nicht verhindern (siehe 03_missbrauch). Ein Versehen bleibt höchstens 20 min als 🟠 sichtbar, mit Hinweis „unbestätigt“.

### T-09 Verlässt sich zu sehr auf die App (Automatisierungs-Vertrauen)
- **Lösung (Schadensbegrenzung):**
  1. **Wording-Katalog als Teil des Designsystems:**
     - Erlaubt: „Keine Meldung“, „Keine Daten“, „Stand unbekannt“.
     - Verboten: „frei“, „sicher“, „ruhig“, „alles klar“, Grün, Häkchen beim Zugstatus.
     - Keine Serien und keine Statistiken wie „12 Fahrten ohne Kontrolle“ oder „X € gespart“.
  2. **Abdeckung und Grundrate (T-02) direkt am ⚪:** „Keine Meldung · wenige Nutzer an Bord · hier oft kontrolliert (Mo–Fr 7–9)“.
  3. **Großkontrollen:** Bei angekündigten Schwerpunkt- oder „Prüfmarathon“-Aktionen (Pressemitteilungen von S-Bahn/hvv) setzt der Betreiber manuell einen Hinweis in `status.json`: „Heute angekündigte Großkontrollen – Meldungen sind dann besonders lückenhaft.“
  4. **Einmaliger Onboarding-Screen mit einem Satz:** „Die App zeigt nur, was andere melden. Große Kontrollen werden oft nicht gemeldet.“ Dazu ein dauerhaft erreichbares „?“ an jedem Status (siehe T-12).
  5. Neutraler Ton gegenüber dem Anstiftungs-Vorwurf (Konzept 8). In „Über die App“ gibt es einen Link zu Ticket-Optionen. Die App gibt keine Texte aus, die zum Fahren ohne Ticket auffordern („Jetzt einsteigen, alles frei“).
- **Status:** ❌ nicht lösbar (im Kern). Eine Kontrolle, die niemand meldet, kann die App nicht sehen. Gerade große, koordinierte Einsätze verhindern das Melden. Die Maßnahmen begrenzen nur die falsche Erwartung.
- **Restrisiko:** Nutzer, die sich trotzdem blind verlassen, werden erwischt. Die App hat dann nie „sicher“ behauptet, und die Datenlücke war sichtbar. Das schützt auch den Betreiber.

### T-10 Mehrere Geräte und doppelte Bestätigungen
- **Lösung:**
  1. **Anonyme Supabase-Anmeldung** (`signInAnonymously`) mit **einmaliger Geräte-Attestierung** bei der Registrierung: Play Integrity bzw. App Attest, geprüft in einer Edge Function. Damit sind die Aufrufe gering, nur einer pro Installation. Das JWT bekommt `attestiert: true`. Web/PWA und Sideload-APK sind nicht attestiert und bekommen Gewicht 0,5.
  2. **Neue Geräte schwach:** Geräte in den ersten 48 h oder ohne bestätigte Meldung tragen zusammen höchstens 1,0 zum Vertrauen einer Meldung bei. Drei frische Geräte machen also nicht mehr als eine Person.
  3. **Korrelation ohne IP-Speicherung:** `melden()` bildet `HMAC(IP, tagessalz)`. Bestätigungen derselben Meldung mit demselben IP-Hash innerhalb von 2 min zählen nur einmal. Das ist ein bewusst konservatives Signal, denn bei CGNAT werden fremde Personen eher zusammengelegt. Dann erscheint ein 🟠 statt 🔴, nie umgekehrt. Der Salz wird täglich gelöscht.
  4. **Bestätigungen nur mit Beleg:** `BESTÄTIGT` für eine Fahrt zählt voll nur aus dem Check-in-Zustand `an_bord` (T-04). Für eine Station gilt der GPS-Nähe-Check aus 5.3.
  5. **Ehrliche Anzeige:** „bestätigt von 3 Geräten“, nie „3 Personen“. 🔴 braucht mindestens 2 voneinander unabhängige, nicht neue Geräte oder 1 eingecheckten Melder.
  6. Entwarnungen löschen nichts (5.3). Das „Freihalten“ einer Strecke durch eine Gerätefarm scheitert damit schon am Prinzip.
- **Status:** 🟡 teilweise.
- **Restrisiko:** Ohne Konten kann ein Mensch mit mehreren echten, attestierten und eingefahrenen Geräten weiterhin mehrfach zählen. Das ist aufwendig und lässt sich mit den Regeln oben auf etwa 2 Stimmen begrenzen. Das Wording „Geräte“ macht klar, dass die App keine Personen zählt.

### T-11 Datenschutzsorge wegen Bewegungs-Tracking
- **Lösung (Datenschutz-Architektur):**
  1. **Tagespseudonym:** `meldungen.geraet` wird ersetzt durch `geraet_tag = HMAC(geraet_id, tagessalz)`. Den Salz erzeugt der Server täglich neu und löscht den alten nach 24 h. Innerhalb eines Tages bleiben Meldungen verknüpfbar (Rate-Limit, Dedupe), über Tage hinweg nicht mehr, auch nicht durch den Betreiber.
  2. **Vertrauenswert getrennt:** `vertrauen(langzeit_hash, score)` mit `langzeit_hash = HMAC(geraet_id, pepper)`, **ohne Zeit, Station oder Fahrt**. Bei einem Leak verrät die Tabelle nur Zahlen.
  3. **Präsenz nicht speichern:** Heartbeats (T-02) liegen höchstens 8 min in einer `unlogged`-Tabelle, danach gibt es nur Buckets. Crowd-Verspätung (6.4) wird ohne Geräte-ID gesendet.
  4. **Kein Hintergrund-Standort:** Bewegungserkennung nutzt nur `ACTIVITY_RECOGNITION` bzw. „Bewegung & Fitness“ und braucht keine Standortberechtigung. GPS gibt es nur „beim Verwenden“ und während des Check-ins: Android über einen Foreground-Service vom Typ `location` mit sichtbarer Benachrichtigung, iOS über die blaue Statusleiste. `ACCESS_BACKGROUND_LOCATION` wird nicht beantragt. Der Store-Text lautet: „Standort nur während einer Fahrt, nur auf dem Gerät“.
  5. **Push ohne Tokens im eigenen System** (FCM-Topics, T-03). Keine Analytics- oder Werbe-SDKs. Crash-Reports nur ohne IDs und nur per Opt-in.
  6. **Hosting:** Supabase-Region EU (Frankfurt), AV-Vertrag abschließen. Plattform-Logs enthalten IPs. Kurze Aufbewahrung beachten und in der Datenschutzerklärung nennen.
  7. **Transparenz:** Die Seite „Was der Server speichert“ zeigt eine Tabelle mit allen Feldern und Löschfristen. Der Quellcode wird veröffentlicht, damit Dritte die Angaben prüfen können. Statistiken gibt es nur aggregiert (Station × Wochentag × Stundenband, Mindestanzahl 5).
  8. **Herausgabe an Behörden:** Nach 24 h gibt es nichts Personenbeziehbares mehr. Das wird in der Datenschutzerklärung genau so gesagt.
- **Status:** ✅ gelöst.
- **Restrisiko:** Zur Laufzeit sieht der Server IP-Adressen und für 24 h die Tagesverknüpfung. Supabase, Google und Apple (Push) sehen Metadaten. Die App benennt das offen und verspricht nicht „anonym“, sondern „ohne Konto, ohne Standortspeicherung, nach 24 h gelöscht“.

### T-12 Farben und Symbole werden falsch verstanden (Barrierefreiheit)
- **Lösung:**
  1. **Form + Text + Farbe, immer alle drei:**

     | Zustand | Form | Text | Farbe |
     |---|---|---|---|
     | Kontrolle | gefülltes Achteck mit „!“ | „Kontrolle“ | Rot |
     | unsicher | Dreieck mit „?“ | „unsicher“ | Gelb-Orange, deutlich hellere Luminanz als Rot |
     | keine Meldung | leerer Kreis | „keine Meldung“ | Grau |
     | keine Daten | gestrichelter Kreis | „keine Daten“ | Hellgrau |
     | Stand unbekannt | Wolke durchgestrichen | „Stand unbekannt“ | Grau |

  2. **Zug und Station visuell trennen:**
     - Züge sind Zug-Glyphen auf einer eigenen Spur links, der Status sitzt als Badge im Glyph.
     - Stationsmeldungen sind ein Etikett rechts neben dem Namen („⚠ Bahnsteig, 09:38“).
     - Der „rote Rand“ für Zugmeldungen (4.2) wird ersetzt durch den Text „↑ eingestiegen 09:38“.
  3. **Kontraste** nach WCAG 2.1 AA: Text 4,5:1, Symbole 3:1. Die Palette wird mit einer Farbfehlsicht-Simulation geprüft (Protan/Deutan). Hell- und Dunkelmodus haben eigene Werte.
  4. **Screenreader:**
     - Keine Emojis in Strings, Icons mit `ExcludeSemantics`.
     - Jede Zeile hat ein `Semantics(label: …)`, z. B. „S1 nach Airport, ab Altona 09:52, in 13 Minuten. Kontrolle im Zug gemeldet 09:38 in Blankenese, von 2 Geräten bestätigt.“
     - Statusänderungen werden über `SemanticsService.announce` angesagt.
  5. **Wischen ersetzbar:**
     - Auf der Zugkarte gibt es zusätzlich die Buttons „‹ Zug davor“ und „Zug danach ›“ sowie `CustomSemanticsAction`s.
     - Die Linienübersicht (4.3) gibt es auch als einfache Liste.
     - Halten-Button (T-08): Mit aktivem Screenreader ersetzt ein Doppeltipp plus Bestätigungsdialog das Halten.
  6. Textskalierung bis 200 % ohne abgeschnittene Inhalte, Tap-Ziele mindestens 48 dp, `reduce motion` beachten. In CI laufen die Flutter-Tests `textContrastGuideline`, `androidTapTargetGuideline` und `labeledTapTargetGuideline`, vor Releases ein manueller TalkBack/VoiceOver-Test.
  7. **Legende:** ein „?“ auf jedem Screen, der Satz „Keine Meldung heißt nicht keine Kontrolle“ steht in der Legende ganz oben.
  8. **BFSG:** Die Pflicht trifft vor allem Personenbeförderungs- und E-Commerce-Dienste. Kleinstunternehmen (unter 10 Beschäftigte und höchstens 2 Mio. € Umsatz) sind bei Dienstleistungen ausgenommen. Eine Pflicht ist daher unwahrscheinlich, wird bei einem Bezahl-Abo aber mit dem Anwalt geklärt (Konzept 8). Ziel ist trotzdem WCAG 2.1 AA bzw. EN 301 549.
- **Status:** ✅ gelöst.
- **Restrisiko:** Die Streifengrafik bleibt visuell dicht. Wer sie nicht lesen kann, bekommt dieselbe Information in der Listenansicht. Eine echte Prüfung mit Betroffenen fehlt noch und gehört in Phase 1.

### T-13 App wird aus dem Store entfernt / Akku leer
- **Lösung Store-Entfernung:**
  1. **Datensatz unabhängig vom Binary:**
     - `sbahn-hh.json` trägt `schema_version`, `gueltig_von` und `gueltig_bis` (aus `feed_info` bzw. `calendar`) sowie eine **Ed25519-Signatur**. Den öffentlichen Schlüssel hat die App fest eingebaut.
     - Die Pipeline erzeugt jede Schema-Version, die noch in Gebrauch ist, mindestens 12 Monate parallel weiter (`v1/`, `v2/`).
     - Ausgeliefert wird von **mehreren Spiegeln** unter eigener Domain: GitHub Pages, Cloudflare Pages, jsDelivr aus dem Repo. Die App probiert sie der Reihe nach.
  2. **Ablauf laut anzeigen:**
     - Nach `gueltig_bis` oder bei einem Datensatz älter als 45 Tage zeigt die App: „Fahrplan veraltet – Zugpositionen können falsch sein“.
     - Zugpositionen und automatische Zugvorschläge werden dann abgeschaltet, Stationsmeldungen gehen weiter.
     - Ist die Schema-Version zu alt: „Bitte App über [Web-Adresse] aktualisieren“.
  3. **Server prüft `datensatz_version`:** Eine GitHub Action lädt die gültigen Fahrten in eine Tabelle `fahrten_gueltig`. Unbekannte `fahrt_key`s ordnet `melden()` über Linie, Station und `beobachtet_um` neu zu (wie in T-05) oder lehnt sie **mit Fehlermeldung** ab, nie still.
  4. **Ausweichwege, schon vor einer Entfernung bekannt gemacht:**
     - PWA unter eigener Domain (Flutter Web, Web Push ab iOS 16.4 für Home-Screen-Apps). Einschränkung: keine Bewegungserkennung, GPS nur im Vordergrund.
     - Android-APK zum Download sowie ein F-Droid-Flavor ohne FCM/Play-Dienste.
     - iOS in der EU optional über einen alternativen Marktplatz.
     - Hinweis „Falls die App aus dem Store verschwindet: [Adresse]“ in den Einstellungen und im Onboarding. `status.json` kann im Ernstfall einen Umzugshinweis an alle bestehenden Installationen ausspielen.
  5. Folgen für T-10: Sideload- und Web-Clients sind nicht attestiert (Gewicht 0,5). Nach einer Entfernung tragen Rate-Limit und Reputation die Hauptlast.
- **Lösung Akku:**
  1. **Akku-Budget:** im Check-in höchstens 3 % pro Stunde, ohne Check-in nahe 0. Das wird in Phase 2 auf je einem Android- und iOS-Gerät gemessen.
  2. Primär die stromsparende Bewegungserkennung. GPS nur mit `balanced`/`reduced accuracy` alle 30–60 s, gezielt um planmäßige Halte. In `funkloch`-Abschnitten (T-01) ist GPS aus. Kein Wakelock, der Bildschirm darf ausgehen. Der Realtime-Socket ist im Hintergrund geschlossen.
  3. **Automatischer Check-out** an der Endstation bzw. spätestens nach 2 h.
  4. **Akku-Stufen** (`battery_plus`, OS-Stromsparmodus):
     - Bei ≤ 15 % oder Stromsparmodus sind GPS und Bewegungserkennung aus, Banner „Akkusparmodus: nur manuelles Melden“.
     - Bei ≤ 5 % checkt die App **aktiv aus** und sendet `MELDER_RAUS` mit Grund `akku`. Der Zug wird ehrlich 🟠, statt dass der Melder still verschwindet.
     - Einmaliger Hinweis: „Denk an dein Handyticket – die App schaltet bei wenig Akku die Sensoren ab.“
- **Status:** 🟡 teilweise. Akku ✅. Bei der Store-Entfernung bleibt die Funktion erhalten, die Reichweite schrumpft.
- **Restrisiko:** Nach einer Entfernung finden neue Nutzer die App kaum noch, und die PWA ist bei Bewegungserkennung und Push schwächer. Alte Installationen bleiben durch Datensatz-Updates korrekt oder melden klar, dass sie veraltet sind. Sie rechnen nie still mit dem alten Fahrplan.

---

## Querschnitt – die wichtigsten Änderungen am Konzept

1. **Neues Meldungs-Ereignismodell mit persistenter Offline-Queue** (KONZEPT §7 Datenmodell, §5.1, §3 „Stabile Fahrt-ID“). `erstellt` wird ersetzt durch `beobachtet_um` (Serverzeit beim Tippen) und `empfangen_um`. Neu sind `client_event_id unique`, `zeitquelle`, `station_id` + `naechste_station_id` zum Beobachtungszeitpunkt, `datensatz_version` und der Typ `ZURUECKGEZOGEN`. Geschrieben wird über die RPC `melden()` mit Plausibilitätsprüfung und Neuzuordnung des `fahrt_key`. Alle Timer in §5.2 laufen ab `beobachtet_um`. → T-01, T-05, T-07, T-08, T-13
2. **Eine zentrale Zeitbasis: Serverzeit + monotone Uhr** (KONZEPT §6.2, Pseudocode `jetzt`). Kein `DateTime.now()` im App-Code, GNSS-Zeit als Fallback, UTC intern, GTFS-Service-Tag mit `Europe/Berlin`. → T-05, T-07, T-01, T-03
3. **Vier ehrliche Zustände + Datenfrische + Fail-loud** (KONZEPT §4.1–4.3, §5.2). Neben 🔴/🟠/⚪ gibt es ◌ „keine Daten“ und „? Stand unbekannt“. Die Verbindungsleiste („Live“ / „Stand vor N min“ / „Offline“) steht auf jedem Screen. Sichtbar sind der Queue-Status, der Tunnel-Hinweis und `status.json` für den Notbetrieb. Dazu kommt ein verbindlicher Wording-Katalog (nie „frei/sicher“, kein Grün) und die Barrierefreiheit Form + Text + Farbe mit Semantics-Labels. → T-01, T-02, T-06, T-07, T-09, T-12
4. **Abdeckungsanzeige aus flüchtiger Präsenz** (KONZEPT §5.3, §7, neu als §5.5). Heartbeats eingecheckter Geräte bleiben 8 min, danach werden sie zu Buckets („wenige/einige/viele“) pro Fahrt und Abschnitt verdichtet. Ab 30 min ohne Beobachter wird der Abschnitt schraffiert. Dazu kommt die Grundrate aus den Aggregaten. Erst damit bedeutet ⚪ etwas. → T-02, T-09, T-01, T-10
5. **Check-in als Zustandsmaschine mit Akku-Budget** (KONZEPT §6.3, §6.5). Die Zustände sind `geplant → an_bord → evtl_raus → raus`. Es braucht 2 Signale und eine Rückfrage, die Haltezeit und Ohlsdorf gelten als Wechselfenster. Bei wenig Akku checkt die App aktiv mit `MELDER_RAUS(akku)` aus. Bestätigungen zählen voll nur aus `an_bord`. → T-04, T-08, T-10, T-13
6. **Datenschutz- und Betriebsarchitektur, unabhängig von Store und Free-Tier** (KONZEPT §6.1, §7, §8, §10 Kosten).
   - Datenschutz: Tagespseudonym per HMAC und täglich gelöschtem Salz, getrennte Vertrauens-Tabelle, Präsenz nie gespeichert, FCM-Topics statt Tokens, kein Hintergrund-Standort, Attestierung nur einmal bei der Registrierung.
   - Datensatz: signiert, versioniert und mit Ablaufdatum, auf mehreren Spiegeln, dazu PWA/APK als vorab kommunizierter Ausweichweg.
   - Betrieb: Writes über REST statt Realtime, Nachladen nach Reconnect, Kanäle pro Linie, Pro-Tarif mit Spend-Cap als eingeplanter Kostenposten.
   - → T-06, T-11, T-13, T-03, T-10
