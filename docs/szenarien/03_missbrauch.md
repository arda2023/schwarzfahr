# Szenarien M: Missbrauch und Angriffe

### M-01 Falsche Entwarnung durch Kontrollteam
- Ablauf: Ein Kontrollteam steigt um 08:20 in Blankenese in die S1 Richtung Airport. Ein Fahrgast meldet IM_ZUG. Ein Kontrolleur hat die App selbst installiert, checkt im selben Zug ein und meldet in Hochkamp AUSGESTIEGEN_IN und NICHT_MEHR_DA. Ein Kollege mit einem zweiten Gerät bestätigt das.
- App vs. Wirklichkeit: Die App zeigt „AUSGESTIEGEN_IN Hochkamp“. Damit wird der Zug neutral ⚪ und Hochkamp rot. Tatsächlich sind die Kontrolleure noch im Zug und kontrollieren bis Altona. AUSGESTIEGEN_IN setzt den Zug laut 5.2 direkt auf neutral und wirkt damit stärker als NICHT_MEHR_DA, das nur Vertrauen abzieht. Die Regel „Entwarnung löscht nichts“ greift hier nicht.
- Folge: Wer in Altona und Königstraße einsteigt, verlässt sich auf die Entwarnung und wird erwischt. Die App verliert an Glaubwürdigkeit.
- Schwere: hoch

### M-02 Hinterhalt durch gelenkte Ausweichbewegung
- Ablauf: Freitag 17:30. Mehrere Geräte melden AN_STATION am Hbf auf allen Linien und dazu IM_ZUG in der S3 Richtung Hbf. Das Team wartet aber an der Sternschanze und in Dammtor, wohin Fahrgäste ausweichen (S11/S21 oder Umstieg).
- App vs. Wirklichkeit: Am Hbf ist alles rot, die Ausweichstationen sind ⚪. Der Hbf ist in Wahrheit frei, die Kontrolle steht dort, wohin die App die Leute schickt.
- Folge: Die Kontrolle wird durch die App gezielter. Weil Fahrgäste ⚪ als „sicher“ lesen (obwohl es nur „keine Meldung“ heißt), haben sie am Ende den Schaden.
- Schwere: hoch

### M-03 Troll-Flut mit vielen Geräten oder Emulatoren
- Ablauf: Eine Person oder ein Discord-Server lässt an einem Abend viele Emulator-Instanzen laufen und setzt jedes Mal die App-Daten zurück, um eine neue anonyme Geräte-ID zu bekommen. Das GPS wird auf die jeweilige Station gefälscht. So kommen auf allen Linien Meldungen im 10-Minuten-Takt pro Instanz, und die Fake-Geräte bestätigen sich gegenseitig.
- App vs. Wirklichkeit: Jeder Zug und jede Station ist 🔴, obwohl nirgends kontrolliert wird. Das Rate-Limit pro Gerät und der Nähe-Check auf dem Gerät halten nichts auf, weil IDs nichts kosten und das GPS vom Client kommt.
- Folge: Die App ist nicht mehr zu gebrauchen und die Leute steigen aus. Dazu kommen höhere Kosten oder das Supabase-Free-Tier-Limit, also ein Ausfall.
- Schwere: hoch

### M-04 Vertrauenswert aufbauen und dann ausnutzen (Sleeper)
- Ablauf: Ein Angreifer (Kontrolldienst oder Troll) macht wochenlang mit einigen Geräten echte Meldungen oder bestätigt die Meldungen anderer und sammelt so hohes Vertrauen. Zu einem gewählten Zeitpunkt, etwa bei einer Schwerpunktaktion, schicken diese Geräte gleichzeitig Entwarnungen oder Ablenkungsmeldungen.
- App vs. Wirklichkeit: Meldungen mit hohem Gewicht schieben echte Warnungen sofort unter die Schwelle (🔴 wird 🟠 oder ⚪). Die Wirklichkeit ist umgekehrt.
- Folge: Nutzer werden genau dann getäuscht, wenn es am meisten zählt. Weil die Geräte anonym sind, findet man den Angreifer später kaum.
- Schwere: hoch

### M-05 Verkehrsunternehmen wertet Kontrollmuster und Ausweichverhalten aus
- Ablauf: Der Sicherheitsdienst liest die öffentlichen Realtime-Kanäle bzw. die Supabase-API mit, denn der Anon-Key steckt in der App. Er speichert alle Meldungen über Monate, auch über die 24 Stunden hinaus, und vergleicht sie mit den eigenen Einsatzplänen.
- App vs. Wirklichkeit: Die App verspricht kurzlebige Daten und eine Löschung nach 24 Stunden. Beim Mitleser liegt ein dauerhaftes Archiv.
- Folge: Das Unternehmen sieht, wie schnell gemeldet wird, wo es keine Melder gibt und wohin Fahrgäste ausweichen. Daraus plant es effektivere Kontrollen, etwa Teamwechsel kurz nach Meldungen oder Kontrollen in Lücken ohne Melder. Die Community hat dem Gegner damit selbst Daten geliefert.
- Schwere: mittel

### M-06 Melder werden über Geräte-ID und Zeitpunkt identifiziert
- Ablauf: Ein Kontrolleur sieht in der S1, wie eine Person um 09:31 in Hochkamp aufs Handy tippt. Sekunden später erscheint IM_ZUG mit Geräte-ID X. Über die Geräte-ID im Realtime-Stream lassen sich die bisherigen Meldungen von X verknüpfen: jeden Werktag 07:50 Rissen, Check-in in die S1, Crowd-Verspätung.
- App vs. Wirklichkeit: Die App sagt „anonym“. Tatsächlich ist die dauerhafte Geräte-ID zusammen mit Station und Zeit ein Pseudonym, aus dem sich ein Bewegungsprofil bauen lässt, auch ohne GPS-Rohdaten.
- Folge: Der Melder kann wiedererkannt werden. Möglich sind gezielte Kontrollen, Hausverbot oder rechtliche Schritte. Die Daten über Wohnort und Wege betreffen nach DSGVO auch Unbeteiligte.
- Schwere: hoch

### M-07 Datenleck oder Behördenanfrage
- Ablauf: Eine Fehlkonfiguration (RLS fehlt, Löschjob fällt still aus) oder eine Anfrage von Staatsanwaltschaft oder Zivilgericht im Rahmen einer Klage wegen Beihilfe oder Unterlassung. Supabase speichert Logs mit IP und Zeitstempel, auch wenn die eigenen Tabellen keine IP enthalten. Push-Tokens aus Phase 3 und der Telegram-Bot mit Nutzernamen kommen dazu.
- App vs. Wirklichkeit: Laut Konzept gibt es keine IP-Speicherung. Bei Hoster, Push-Dienst und Telegram fallen aber Metadaten an, über die sich Personen identifizieren lassen.
- Folge: Melder werden identifiziert. Der Betreiber muss Daten herausgeben und haftet womöglich, Verein oder Privatperson mit Impressum ist dabei ein eigenes Risiko.
- Schwere: hoch

### M-08 Konkurrenz sabotiert
- Ablauf: Kurz vor dem Start auf r/hamburg setzt ein konkurrierender Anbieter Fake-Meldungen ab, schreibt Ein-Stern-Bewertungen („zeigt nur Unsinn“) und meldet die App bei Apple und Google wegen „Aufforderung zum Schwarzfahren“. Den Repo-Namen „schwarzfahr“ zitiert er als Beleg.
- App vs. Wirklichkeit: In der ersten Woche gibt es wenige echte Meldungen und viele falsche. Der Ruf ist schnell ruiniert.
- Folge: Der Kaltstart scheitert oder die App wird aus dem Store entfernt. Das trifft den Betreiber, und die Nutzer bleiben dann bei der Web-Version.
- Schwere: mittel

### M-09 Spaßmeldungen von Schülergruppen
- Ablauf: Eine 8. Klasse fährt um 13:15 S3 von Neugraben nach Harburg. Zehn Schüler mit eigenen Handys sitzen im selben Zug und sind dort eingecheckt, der Nähe-Check ist also echt erfüllt. Zum Spaß melden sie IM_ZUG und bestätigen sich gegenseitig, danach AN_STATION in Harburg.
- App vs. Wirklichkeit: Der Zug ist sehr vertrauenswürdig 🔴, das sieht aus wie ein bestätigter Fund. Kontrolleure gibt es keine.
- Folge: Andere Fahrgäste steigen grundlos aus oder lassen Züge fahren, kommen zu spät und verpassen Anschlüsse. Die Vertrauenswerte werden verfälscht, denn die Schüler bestätigen sich gegenseitig und ihre Werte steigen. Es entsteht Alarmmüdigkeit.
- Schwere: mittel

### M-10 Gefährliche Flucht und Gedränge
- Ablauf: Eine Push-Meldung „Kontrolle in deinem Zug“ (Phase 3) kommt kurz vor dem Halt an der Landungsbrücken. Mehrere Fahrgäste drängen gleichzeitig zur Tür, einer springt kurz vor dem Schließen heraus. Andere rennen am Bahnsteig vom Zug weg oder über die Treppe.
- App vs. Wirklichkeit: Die Meldung kann ein Fake sein oder dem falschen Zug zugeordnet sein (Verspätung, Flügelung). Die Reaktion in Echtzeit ist trotzdem real.
- Folge: Es kann Stürze und Verletzungen am Bahnsteig geben. Gefährdet sind Fahrgäste und auch Kontrolleure, die dabei angerempelt werden.
- Schwere: hoch

### M-11 Belästigung von Kontrolleuren
- Ablauf: In einer Telegram-Gruppe, die App-Meldungen per Bot spiegelt, posten Nutzer zusätzlich Fotos und Beschreibungen, etwa „die Blonde mit der roten Jacke, jeden Dienstag S21 Bergedorf“. Mit dem Zug-Tracking finden einzelne Personen das Team, folgen ihm und filmen.
- App vs. Wirklichkeit: Die App selbst hat keinen Freitext und keine Fotos. Mit dem Bot und den genauen Zugpositionen wird sie aber zum Werkzeug, um einzelne Personen zu verfolgen, außerhalb ihrer eigenen Regeln.
- Folge: Kontrolleure werden bedroht oder angegriffen. Für den Betreiber folgen Mitverantwortung, rechtliche Schritte, Presseberichte und der Store-Ausschluss.
- Schwere: hoch

### M-12 Verspätungsdaten gefälscht, Meldungen landen im falschen Zug
- Ablauf: Ein Troll schickt für alle S1-Fahrten in der HVZ Crowd-Verspätungen von +5 Minuten, im 5-Minuten-Takt. Alle Handys verschieben ihre Zugpositionen entsprechend.
- App vs. Wirklichkeit: Wer nicht eingecheckt ist, meldet am falschen Zug, nämlich am Vorgänger oder Nachfolger. Ein echter 🔴 sitzt dann auf dem Zug, der schon weg ist, und der Zug, der gleich einfährt, zeigt ⚪.
- Folge: Echte Warnungen verpuffen leise, und Nutzer sind gefährdet, ohne dass die Manipulation auffällt. Weil die Abfahrtstafel falsch ist, leidet auch der Nutzen der App selbst.
- Schwere: mittel

### M-13 Verwechslung bei der Flügelung in Ohlsdorf
- Ablauf: Ein Fahrgast im hinteren Teil der S1 meldet versehentlich „vorne“, oder ein Troll setzt den Zugteil absichtlich falsch. In Ohlsdorf geht die Meldung an den Airport-Zug, die Kontrolleure fahren aber nach Poppenbüttel.
- App vs. Wirklichkeit: Der Airport-Ast ist rot und der Poppenbüttel-Ast ⚪, in Wahrheit ist es umgekehrt.
- Folge: Fahrgäste nach Poppenbüttel werden in falscher Sicherheit gewiegt. Wer zum Flughafen will, steigt grundlos aus und verpasst womöglich seinen Flug.
- Schwere: niedrig bis mittel

Aus den Szenarien abgeleitete Schwachstellen im Konzept:
- AUSGESTIEGEN_IN wirkt als ungeprüfte Entwarnung (siehe 5.1/5.2).
- Geräte-IDs kosten nichts, und der Nähe-Check läuft nur auf dem Client.
- Die Geräte-ID ist dauerhaft und im Realtime-Stream öffentlich sichtbar.
- Ein öffentlicher Lesezugriff macht die Löschung nach 24 Stunden wirkungslos.
- Crowd-Verspätungen werden nicht geprüft.
- Bei Hoster, Push-Dienst und Telegram fallen Metadaten an.
