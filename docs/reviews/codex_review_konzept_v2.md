# Review von Codex zu Konzept Version 2

Stand: 3. November 2026. Eingefügt vom Nutzer, unverändert bis auf die Formatierung. Bezieht sich auf Commit `7079d44`.

---

**Einschätzung: Der Kern ist umsetzbar. Die größte Schwierigkeit ist, eine Beobachtung zuverlässig dem richtigen Zug zuzuordnen und Unsicherheit verständlich darzustellen.** Das Konzept behandelt viele Sonderfälle, erklärt aber einige davon zu früh für „gelöst“. Außerdem würde ich den ersten Funktionsumfang deutlich verkleinern.

Die Bewertung ist eine Konzeptprüfung. Die Verkehrs- und API-Daten aus der Recherche wurden nicht unabhängig im Livebetrieb verifiziert.

## 1. Idee und Nutzen

Community-App für zunächst die Hamburger S-Bahn: Abfahrten an der Station, einzelne Züge auf einem Stationsstreifen, Meldungen über Kontrollen in einem Zug, Zugteil oder an einer Station, Zeitpunkt, Ort und Verlässlichkeit der letzten Beobachtung. Besonderer Ansatz: Zuordnung zu einzelnen Fahrten. Dazu Alltagsnutzen (Abfahrten, Verspätungen, Stationsverlauf), wichtig für den Start.

„Das gibt es nirgends“ ist eine noch zu belegende Marktannahme.

## 2. Bedienung

1. App öffnen: Station über Favoriten oder Standort, Abfahrtstafel mit Meldungen und Datenstand.
2. Zugkarte: Stationsverlauf, geschätzte Position, letzte Beobachtung; andere Züge über Wischen und Buttons.
3. Einsteigen bestätigen: Linie, Station und Ziel am Zug; bei Mehrdeutigkeit bleibt die Zuordnung ausdrücklich unsicher.
4. Melden: „Im Zug“ oder „An der Station“, danach Zugteil oder Ortsangabe ergänzen.
5. Während der Fahrt: kurze Rückfragen, Korrektur, Ausstieg bestätigen.
6. Funkloch: letzter bekannter Stand, nicht versendete Meldungen gekennzeichnet.

Lesen ohne Registrierung, zum Schreiben eine pseudonyme Sitzung im Hintergrund. Den Zwei-Tap-Ablauf nur bei bereits bekanntem Zug bzw. bekannter Station versprechen.

## 3. Technologieempfehlung

| Bereich | Empfehlung |
|---|---|
| Webapp/PWA | React + TypeScript + Vite |
| Mobile Apps | Flutter + Dart |
| Backend | Supabase mit PostgreSQL in einer EU-Region |
| Fachlogik | TypeScript auf dem Server (Supabase Edge Functions) |
| Fahrplandaten | Python-Import für GTFS |
| Echtzeitdaten | geplanter Hintergrundjob mit gemeinsamem Cache |
| Statische Auslieferung | z. B. Cloudflare Pages und Objektspeicher/CDN |
| Offline-Speicherung | IndexedDB im Web, SQLite in Flutter |

Web und Flutter teilen sich API-Verträge und Datenformate, die Oberflächen werden separat entwickelt.

**Wichtigste Architekturänderung: Der Server berechnet den verbindlichen Meldungsstatus** (statt jedes Handy aus bereinigten Einzelereignissen). Dadurch zeigen Web und Mobil denselben Zustand, Modelländerungen brauchen kein App-Update, und Clients erhalten keine Einzelmeldungen, aus denen sich Bewegungsmuster rekonstruieren lassen. Zugpositionen können weiter lokal animiert werden.

## 4. Bewertung der Szenarien

Das Repository nennt 21 gelöst, 42 teilweise, 4 nicht lösbar. Mehrere grüne Bewertungen sollten zurückgestuft werden.

| Szenarien | Bewertung | Kommentar |
|---|---|---|
| S-01, S-05, S-06, K-07 (Meldung altert) | sinnvoll abgefedert | Aufenthaltsort bleibt ohne neue Beobachtung unbekannt. |
| S-02, S-10, S-11, K-01, T-02, T-09 (fehlende Meldungen) | grundsätzlich begrenzt | „Keine Meldung“ und „keine aktuellen Beobachtungen“ unterscheidbar machen. |
| S-07, S-14, Z-01, Z-09–Z-11 (Verspätung, falsche Linie, mehrere Züge) | **zentrales ungelöstes Risiko** | Zwei verspätete Züge mit identischem Ziel bleiben u. U. ununterscheidbar, auch mit Echtzeit. |
| S-13, Z-02, Z-03, Z-13 (Ausfälle, Kurzwenden, SEV) | teilweise lösbar | Fehlende Daten dürfen nicht als reguläre Fahrt erscheinen. |
| S-03, Z-04–Z-08, M-13 (Kopplungen) | gutes Datenmodell, zu starke Annahmen | Gleiche Zeiten beweisen keine Kopplung; das Ziel beweist nicht den Zugteil. |
| S-09, K-02, K-08 (lange Züge) | teilweise lösbar | Entwarnung aus dem vorderen Teil darf den hinteren nicht entwarnen. Erwartete ≠ bestätigte Zuglänge. |
| S-04, S-08, Z-15, K-03, K-05 (Umstieg, Pendeln) | Vorhersagen zu spekulativ | Alte Meldungen sollten nicht automatisch verschwinden. |
| K-04, K-06 (Abgangs- und Großkontrollen) | darstellbar, nicht vollständig gelöst | Feste Mindestdauer von 90 min erzeugt lange Fehlwarnungen. |
| S-12, K-10–K-12 (Irrtümer) | teilweise lösbar | „Nur gesehen“ darf keine bestätigte Kontrolle ergeben. |
| M-01–M-04, M-08, M-09, M-12, K-09, T-10 (Fakes) | erschwerbar, nicht verhinderbar | Mehrere Geräte sind kein Beweis für mehrere Personen. |
| M-05–M-07, T-11 (Mitlesen, Identifikation) | reduzierbar | Öffentliche Zustandsdaten statt Einzelereignissen. |
| T-01, T-05–T-07, Z-14 (Funkloch, Uhr, Cache) | technisch gut beherrschbar | Details zur Synchronisierung fehlen. |
| Z-12, T-03, T-04, T-08, T-12, T-13, M-10, M-11 | unterschiedlich | PWA braucht einen vollständigen manuellen Ablauf. |

## 5. Korrekturen am Konzept

- **Wahrscheinlichkeitsmodell inkonsistent:** `0,85 × Gerätegewicht 1,5 = 1,275`. Für den Start lieber nachvollziehbare Zustände („einzeln gemeldet“, „mehrfach bestätigt“, „widersprüchlich“, „veraltet“), Statistik später.
- **Anwesenheit ist keine Bestätigung:** Ein Heartbeat darf eine Beobachtung nicht auffrischen. Abgebrochener Heartbeat = „Präsenz unbekannt“, nicht „ausgestiegen“.
- **Getrennte Vertrauenswerte** für Beobachtung und Zuordnung zum Zug.
- **Fahrt-ID nicht dauerhaft stabil:** besser Datenquelle + Datensatzversion + Fahrt-ID + Betriebstag, dazu eine Zuordnung zwischen Versionen. Gleiche Stationen mit verschiedenen Gleisen unterscheidbar halten.
- **„Neue Warnung gewinnt immer“** erlaubt, alte Warnungen beliebig zu verlängern. Wiederholungen derselben Quelle dürfen keine zusätzlichen Stimmen erzeugen.
- **Teamverfolgung aus dem ersten Produkt streichen.**
- **Web braucht einen eigenen Vertrauensweg** (App Attest/Play Integrity gibt es im Web nicht).
- **Verbindliche Regelversion:** Dokumente widersprechen sich (Vibration, Ablaufzeiten, Gewichte, Server- vs. Client-Berechnung).

## 6. Weitere Szenarien

| Szenario | Stärke | Behandlung |
|---|---|---|
| Offline-Meldung kommt nach ihrer Rücknahme an | hoch | Rücknahmen vormerken, Ereignisse verweisen aufeinander. |
| Nach Verbindungsabbruch fehlen Zwischenereignisse | hoch | Zustand mit Versionsnummer nachladen; UUID ist kein Cursor. |
| Datenquelle antwortet erfolgreich, liefert aber alte Daten | hoch | Datenalter je Quelle und Fahrt messen. |
| Neue Nutzer bestätigen sich gegenseitig | hoch | Gegenseitige Bestätigung begrenzen. |
| Echte Großkontrolle löst die Spam-Notbremse aus | hoch | Echte Spitzen nicht unterdrücken, Limits aber nicht pauschal aufheben. |
| Warnung bleibt durch ständige Bestätigung wochenlang | hoch | absolute Speicherobergrenzen. |
| Viele teilen Mobilfunk-IP oder Zug-WLAN | mittel | IP nur schwaches Signal. |
| Browser geschlossen / OS beendet App | hoch | Präsenz auslaufen lassen, keine Hintergrundüberwachung versprechen. |
| Fahrplanwechsel oder Zeitumstellung während einer Nachtfahrt | hoch | Betriebstag, Zeiten > 24 Uhr und Datensatzwechsel gemeinsam behandeln. |
| Viraler Beitrag bringt sehr viele Leser | hoch | Zustände cachen, Updates bündeln, Abos begrenzen. |
| Neue Meldung nach korrekter Entwarnung (anderes Team) | mittel | Erste Person verliert keine Reputation. |
| Statistik zeigt vor allem Orte mit vielen Nutzern | mittel | Meldehäufigkeit ≠ Kontrollwahrscheinlichkeit. |

## 7. Betrieb und Kosten

Ein Supabase-Projekt, statisches Webhosting, schlanker Datenjob. Nur relevante Linien abonnieren, Änderungen bündeln, Fahrplandateien über CDN, Animation lokal.

Rechenbeispiel: 1.000 gleichzeitige Nutzer, 8 h/Tag, alle 30 s eine Nachricht ≈ 28,8 Mio. Zustellungen pro Monat.

| Stand | Budget/Monat (Planungsreserve) |
|---|---:|
| Prototyp, geschlossene Testgruppe | ca. 0–30 € |
| kleine öffentliche Beta | ca. 30–100 € |
| Wachstum | nach Nutzern und Nachrichten |

Dazu überwachte Importe, Backups mit geprüfter Wiederherstellung, Fehlerüberwachung, eingeschränkter Betriebsmodus.

## 8. Phasen

| Phase | Inhalt | Kriterium |
|---|---|---|
| 0 | Konzept bereinigen, GTFS und Echtzeitabdeckung der S1 prüfen, Recht früh einordnen | bekannt, welche Fahrten zuverlässig erkennbar sind |
| 1 | React-PWA ohne Community: Station, Tafel, Zugkarte, Linie, simulierte Meldungen | Testpersonen verstehen die Anzeige |
| 2 | Geschlossene Web-Beta S1: Backend, echte Meldungen, Check-in, Korrektur, Offline, zentrale Bewertung, Echtzeit | wichtigste Szenarien funktionieren, Fehler messbar |
| 3 | Flutter-Apps mit derselben API | funktioniert bei Funkloch, Neustart, verweigerten Rechten |
| 4 | Öffentliche Beta vorbereiten | Qualitäts- und Betriebsziele erreicht |
| 5 | Erweitern | nachweisbar bessere Datenqualität |

Teamzuordnung, Umstiegsvorhersagen, historische Kontrollwahrscheinlichkeiten und Push später. Für eine Person eher mehrere Monate als wenige Wochen.

Vorgeschlagener nächster Schritt: bereinigte Version 3 mit React/TypeScript fürs Web, Flutter mobil, zentraler Statusberechnung, engem S1-Umfang und prüfbaren Kriterien. Danach als erste technische Untersuchung die reale Zugzuordnung mit Fahrplan- und Echtzeitdaten.
