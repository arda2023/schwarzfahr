# Lösungen M: Missbrauch und Angriffe

Stand: 3. November 2026 · Bezug: [`03_missbrauch.md`](03_missbrauch.md), [`../KONZEPT.md`](../KONZEPT.md), [`../recherche/konkurrenz_kontrolleur_apps.md`](../recherche/konkurrenz_kontrolleur_apps.md)

Nur Verteidigung auf Konzeptebene. Alle Zahlen sind **Startwerte**, die mit echten Daten nachjustiert werden müssen.

**Faktenkorrekturen zu den Szenarien:**
- M-02: S11 und S21 gibt es seit dem Fahrplanwechsel Dezember 2023 nicht mehr. Dammtor und Sternschanze liegen an der Verbindungsbahn; welche Linien dort fahren, kommt aus dem aktuellen GTFS (nicht hart codieren, KONZEPT 6.1). Das Szenario bleibt inhaltlich gleich.
- M-11: „S21 Bergedorf“ heißt heute S2 (Bergedorf/Aumühle). Der Kern des Szenarios ändert sich nicht.
- M-13: Welcher Zugteil in Ohlsdorf wohin fährt (KONZEPT 5.4: vorne → Airport), muss vor dem Launch vor Ort geprüft und als Daten gepflegt werden, nicht als Konstante.

**Begriffe, die unten verwendet werden:**

| Begriff | Bedeutung |
|---|---|
| **Konto** | Serverseitiger, anonymer Datensatz pro attestierter Installation (kein Account). Schlüssel: `HMAC(server_secret, attest_key_id)`. Enthält **nur** Stufe, Erstellwoche, Zähler, Device-Recall-Flag, **keine Orte und keine Meldungshistorie**. |
| **Stufe** | S0 = neu (< 7 Tage alt oder < 3 unabhängig bestätigte Meldungen), S1 = etabliert, S2 = bewährt (≥ 30 Tage, ≥ 10 unabhängig bestätigte Meldungen). Ohne Attestierung (Web, de-googelte Androids): Stufe W. |
| **Ereignis** | Server-Objekt „Kontrolle in Fahrt X“ oder „Kontrolle an Station Y“, zu dem Meldungen, Bestätigungen und Entwarnungen gehören. |
| **Cluster** | Beiträge zu einem Ereignis, die nicht unabhängig sind: gleiche Station, gleiche Fahrt, innerhalb von 3 Minuten. Ein Cluster zählt höchstens wie 1,5 Beiträge. |
| **Unabhängig** | Andere Konten **und** anderer Cluster (andere Station, ≥ 5 Min. später, oder Zug- vs. Bahnsteigmeldung). |

**Gewichte (Startwerte):**

| Stufe | Warnung (`IM_ZUG`, `AN_STATION`, `BESTÄTIGT`) | Entwarnung (`AUSGESTIEGEN_IN`, `NICHT_MEHR_DA`) |
|---|---|---|
| W (ohne Attest) | 0,25 | 0 (wird nur gezählt und angezeigt) |
| S0 | 0,5 | 0 (wird nur gezählt und angezeigt) |
| S1 | 1,0 | 0,5 |
| S2 | 1,5 (Deckel) | 0,75 (Deckel) |

- 🟠 „gemeldet“: Summe ≥ 0,5. Eine einzelne attestierte Meldung reicht, damit eine Warnung sichtbar ist.
- 🔴 „bestätigt“: Summe ≥ 2,0 **aus ≥ 2 unabhängigen Clustern**.
- Jedes Konto trägt pro Ereignis höchstens einmal bei.

---

### M-01 Falsche Entwarnung durch Kontrollteam
- **Lösung:**
  1. **`AUSGESTIEGEN_IN` setzt den Zug nie direkt auf ⚪** (Änderung an KONZEPT 5.1/5.2). Eine Ausstiegsmeldung macht aus 🔴 höchstens 🟠 „evtl. ausgestiegen in Hochkamp (1×, unbestätigt)“. Die Station Hochkamp wird 🟠, nicht 🔴.
  2. **⚪ nur über diese Wege:** Fahrtende; Zeitablauf ohne neue Warnung (KONZEPT 5.2); oder Entwarnungen mit Summe ≥ 1,0 aus ≥ 2 unabhängigen Konten der Stufe S1/S2 **und** danach 10 Minuten ohne neue Warnung.
  3. **Warnung gewinnt:** Jede neue `IM_ZUG` oder `BESTÄTIGT` nach einer Ausstiegsmeldung setzt den Zug sofort zurück auf mindestens 🟠 bzw. 🔴, und die Ausstiegsmeldung verliert ihr Gewicht.
  4. **Serverseitige Plausibilität:** Eine Ausstiegsmeldung zählt nur, wenn die Fahrt laut Fahrplan (plus Crowd-Verspätung) in diesem Moment an der genannten Station hält oder sie gerade verlassen hat (±2 Min.). Der Server lädt dafür dieselbe `sbahn-hh.json`.
  5. **Neue Geräte entwarnen nicht:** S0- und W-Entwarnungen haben Gewicht 0. Damit wirkt das „zweite Gerät des Kollegen“ nur, wenn es seit Wochen als etabliertes Konto läuft.
  6. **Anzeige:** „Kontrolle gemeldet 08:21 in Blankenese · 1× als ausgestiegen in Hochkamp gemeldet“. Beides bleibt sichtbar (Prinzip MemetroMap).
- **Status:** 🟡 teilweise
- **Restrisiko:** Ein Team, das zwei Konten über Wochen zur Stufe S1 aufbaut, kann einen Zug nach 10 ruhigen Minuten auf ⚪ bringen, wenn danach niemand mehr warnt. Das ist der Preis dafür, dass echte Ausstiegsmeldungen (Kern-Feature) überhaupt wirken. Transparenz: Die Zugkarte zeigt immer die Historie des Ereignisses („gemeldet … · entwarnt …“), und ⚪ heißt weiterhin nur „keine Meldung“.

### M-02 Hinterhalt durch gelenkte Ausweichbewegung
- **Lösung:**
  1. **Keine Ausweich- oder Routenempfehlungen** in der App („Fahr lieber über Dammtor“ o. Ä.). Die App zeigt Meldungen, sie lenkt keine Ströme. Das senkt auch die rechtliche Angriffsfläche (KONZEPT 8).
  2. **🔴 braucht unabhängige Cluster** (siehe Gewichte). Viele gleichzeitige Meldungen junger Konten an einem Ort ergeben höchstens 🟠.
  3. **Anomalie-Erkennung serverseitig:** Wenn an einem Knoten (Hbf, Altona, Jungfernstieg) in kurzer Zeit deutlich mehr Meldungen eingehen als im Mittel und der Großteil von S0/W-Konten stammt, bleibt das Ereignis 🟠 mit dem Zusatz „viele neue Melder, ungeprüft“.
  4. **Wording für ⚪:** Der Text heißt „keine Meldung“ und beim Antippen „Kontrollen können überall stattfinden, auch ohne Meldung“. Es gibt kein Grün (bereits KONZEPT 5.2).
- **Status:** 🟡 teilweise
- **Restrisiko:** Stehen echte Menschen mit etablierten Konten tatsächlich am Hbf und melden falsch, kann die App das nicht von einer echten Kontrolle unterscheiden. Eine Anzeige der Abdeckung („auf dieser Linie gerade wenige Melder“) würde Fahrgästen helfen, verrät aber dem Verkehrsunternehmen die Lücken (M-05). Deshalb höchstens netzweit und grob („Datenlage heute dünn“). Transparenz: In den FAQ steht ausdrücklich, dass Meldungen gefälscht sein können und die App keine Sicherheit verspricht.

### M-03 Troll-Flut mit vielen Geräten oder Emulatoren
- **Lösung:**
  1. **Schreiben nur mit Attestierung:** Bei der ersten Meldung erzeugt die App einen Schlüssel im Secure Enclave/Keystore und lässt ihn über **App Attest (iOS)** bzw. **Play Integrity (Android)** bestätigen. Eine Edge Function prüft das und legt das Konto an. Jede Meldung trägt ein kurzlebiges Token plus Assertion. Emulatoren und veränderte Apps fallen damit weitgehend heraus.
  2. **Neuinstallation kostet etwas:** **DeviceCheck** (iOS, 2 Bits pro Gerät) bzw. **Play Integrity Device Recall** (Android, sofern verfügbar) merken sich über Neuinstallationen hinweg nur „auf diesem Gerät gab es schon ein Konto“. Ein neues Konto auf so einem Gerät startet ohne Alters-Bonus und bleibt länger in S0. Die Bits identifizieren niemanden.
  3. **Alter zählt:** S0-Konten können warnen (🟠), aber nicht allein 🔴 erzeugen und nicht entwarnen.
  4. **Alle Prüfungen serverseitig:** Rate-Limit pro Konto (1 neue Meldung / 10 Min., 1 Beitrag pro Ereignis), Fahrplan-Plausibilität (Fahrt existiert, ist unterwegs und hält an der Station), ein Konto kann nur in **einer** Fahrt gleichzeitig aktiv sein. Der GPS-Nähe-Check bleibt auf dem Gerät, wirkt aber erst zusammen mit der Attestierung (die geprüfte App hat ihn durchgeführt). Auf Android zusätzlich das Mock-Location-Flag des Systems als Signal mitsenden.
  5. **Grobe Netz-Limits ohne Speicherung:** Ein IP-Zähler nur im Arbeitsspeicher der Edge Function (z. B. 30 Schreibanfragen/Min.). Wegen CGNAT im Mobilfunk großzügig wählen. Anfragen aus Rechenzentrumsnetzen erhalten Gewicht 0.
  6. **Notbremse:** Steigt die Zahl der Meldungen pro Linie und Stunde über das 5-Fache des gleitenden Mittels, nimmt der Server für 60 Minuten nur noch Meldungen von S1/S2 an. Die App zeigt das ehrlich: „Ungewöhnlich viele Meldungen, nur geprüfte Melder werden angezeigt.“
  7. **Kosten und Ausfall:** Lesen läuft nicht über die Datenbank, sondern über einen zwischengespeicherten Zustands-Snapshot (CDN, 15 s) und Realtime-Broadcast (siehe M-05). Schreibversuche ohne gültiges Token werden in der Edge Function abgewiesen, bevor die Datenbank etwas tut. Budget-Alarm beim Hoster.
  8. **Web und de-googelte Androids** (GrapheneOS u. a., die Attestierung oft nicht bestehen, aber gerade bei Datenschutz-Bewussten verbreitet sind) bekommen Stufe W: Lesen voll, Warnen mit Gewicht 0,25, kein Entwarnen. Zusätzlich ein Captcha ohne Tracking (z. B. Turnstile oder Private Access Tokens).
- **Status:** 🟡 teilweise
- **Restrisiko:** Eine Farm echter, nicht manipulierter Handys mit Zeitaufwand umgeht Attestierung und Alter. Das kostet aber Geld und Wochen statt Minuten. Attestierung beweist nur „echte App auf echtem Gerät“, **nicht** den Aufenthaltsort; echtes Proof-of-Presence ist ohne vertrauenswürdige Infrastruktur im Zug (Beacons des Betreibers) nicht möglich. Apple und Google sehen bei jeder Attestierung, dass die App genutzt wird. Transparenz: Notbremsen-Hinweis in der App, Datenschutzerklärung nennt Attestierung.

### M-04 Vertrauenswert aufbauen und dann ausnutzen (Sleeper)
- **Lösung:**
  1. **Gewicht mit Deckel:** Vertrauen erhöht das Gewicht nur bis 1,5 (Warnung) bzw. 0,75 (Entwarnung). Hoher Ruf bringt also wenig zusätzliche Macht.
  2. **Asymmetrie:** Entwarnungen können 🔴 nie direkt auf ⚪ setzen (siehe M-01). Ein Sleeper-Angriff erzeugt höchstens 🟠 „widersprüchlich“, und die Anzeige zeigt beide Seiten („3× gemeldet · 2× entwarnt“).
  3. **Vertrauen nur durch Fremde:** Der Zähler „unabhängig bestätigte Meldungen“ steigt nur, wenn ein Konto aus einem **anderen** Cluster bestätigt. Gegenseitiges Bestätigen in der eigenen Gruppe bringt nichts.
  4. **Vertrauen verfällt:** Ohne Aktivität sinkt die Stufe nach 30 Tagen um eins. Jede Entwarnung, auf die innerhalb von 15 Min. eine neue unabhängige Warnung derselben Fahrt folgt, setzt das Konto auf S0 zurück. Diese Regel ist für den Angreifer besonders teuer.
  5. **Korrelations-Bremse:** Kommen in kurzer Zeit auf mehrere Ereignisse Entwarnungen von vielen S2-Konten gleichzeitig (deutlich über dem Normalwert), gelten Entwarnungen netzweit 30 Minuten lang mit Gewicht 0, und die App sagt das.
- **Status:** 🟡 teilweise
- **Restrisiko:** Ein geduldiger Angreifer mit vielen echten Konten kann echte Warnungen in 🟠 verwandeln und damit Unsicherheit stiften. Ein falsches ⚪ ist nur eingeschränkt möglich (siehe M-01). Die Zurückverfolgung ist wegen der gewollten Anonymität tatsächlich kaum möglich. Das ist ein bewusster Zielkonflikt mit M-06. Transparenz: „widersprüchlich“ als eigener Text statt stillem Farbwechsel.

### M-05 Verkehrsunternehmen wertet Kontrollmuster und Ausweichverhalten aus
- **Lösung:**
  1. **Kein Lesen der Rohdaten:** RLS sperrt `select` auf `meldungen` für `anon` und `authenticated` vollständig. Kein `postgres_changes` auf der Tabelle. Schreiben nur über die RPC/Edge Function `melden()`.
  2. **Ausgeliefert wird nur der aggregierte Zustand:** pro Fahrt und Station Status (⚪/🟠/🔴), Ort der ersten Meldung, Zeit auf die Minute gerundet, Anzahl der Meldungen in Stufen („1“, „2–3“, „mehr“). Keine Geräte-IDs, keine Einzelzeitpunkte, keine Gewichte.
  3. **Keine Historie über die API:** Es gibt nur „jetzt“, keinen Endpunkt für frühere Zustände.
  4. **Rohmeldungen kürzer halten als 24 h:** Löschung beim Ende des Ereignisses + 2 Stunden. Danach bleiben nur grobe Statistiken (Station × Wochentag × 4 Tagesabschnitte, nur veröffentlicht ab 5 Ereignissen pro Zelle).
  5. **Lesen nur über attestierte App oder Web mit Captcha** und mit Rate-Limit. Das erschwert Massenabfragen, verhindert sie aber nicht.
- **Status:** ❌ nicht lösbar (im Kern), aber deutlich reduzierbar
- **Restrisiko:** Was ein Fahrgast sieht, sieht auch ein Mitarbeiter mit einem normalen Handy. Ein Dauerabruf des Zustands genügt, um ein Archiv aufzubauen und Reaktionszeiten, Lücken und Ausweichverhalten auszuwerten. Die Maßnahmen schützen die Melder (keine Rohdaten, keine IDs), nicht die Information an sich. Transparenz: In der App und den FAQ steht klar: „Alles, was die App anzeigt, kann jeder sehen und speichern, auch Verkehrsunternehmen. Die Löschfrist gilt für unseren Server, nicht für Mitlesende.“ Die Formulierung „Löschung nach 24 h“ darf nicht als Versprechen für die angezeigten Daten erscheinen.

### M-06 Melder werden über Geräte-ID und Zeitpunkt identifiziert
- **Lösung:**
  1. **Keine Geräte-ID im öffentlichen Stream** (folgt aus M-05). Die Spalte `geraet` ist für Clients nicht lesbar.
  2. **Entkoppelte Pseudonyme:** Eine Meldung speichert die Konto-Referenz nur so lange, wie das Ereignis läuft (Bewertung der Bestätigungen, max. 2 h). Danach wird die Spalte auf `NULL` gesetzt und das Konto bekommt nur seinen Zähler gutgeschrieben. Über Tage gibt es keine Verknüpfung „Konto → Orte“.
  3. **Zufällige Verzögerung beim Veröffentlichen** neuer Ereignisse und Statuswechsel um 10–40 Sekunden, Zeitstempel auf die Minute gerundet. Wer sieht, dass jemand tippt, kann den Farbwechsel nicht mehr sicher dieser Person zuordnen. Zielkonflikt: Eine S-Bahn braucht zwischen zwei Stationen meist 1–3 Minuten, 40 Sekunden Verzögerung kosten also wenig Nutzen.
  4. **Check-in bleibt lokal**, Crowd-Verspätungen werden ohne Konto-Bezug gesendet: Der Server stellt pro Fahrt ein kurzlebiges Fahrt-Token aus, das nur die Stufe des Kontos enthält (für M-12) und für das Rate-Limit dient. Welches Konto das Token bekam, wird nicht gespeichert.
  5. **„Später melden“:** Die Zugmeldung ist auch kurz nach dem Aussteigen noch für die eben verlassene Fahrt möglich (5 Min., gleiche Plausibilitätsprüfung). Wer sich beobachtet fühlt, muss nicht im Zug tippen.
- **Status:** ✅ gelöst (Profilbildung) – Rest siehe unten
- **Restrisiko:** Eine Beobachtung im Moment (eine Person tippt, in einem fast leeren Wagen) lässt sich nie ganz ausschließen. Daraus entsteht aber kein Verlauf mehr. Wer die App auf dem Handy offen hat, gibt sich bei einer Kontrolle als Nutzer zu erkennen; das ist kein Beweis für irgendetwas, die App sollte aber nicht so wirken, als sei sie unsichtbar. Transparenz: Datenschutzseite erklärt in drei Sätzen, was der Server wie lange weiß.

### M-07 Datenleck oder Behördenanfrage
- **Lösung:**
  1. **Was nicht existiert, kann nicht herausgegeben werden:** keine IP in eigenen Tabellen, keine dauerhafte Verknüpfung Konto ↔ Meldung (M-06), Rohmeldungen nur bis Ereignisende + 2 h, HMAC-Schlüssel für Konten nur auf dem Server, Tagesschlüssel werden täglich verworfen.
  2. **Hoster-Logs aktiv klein halten:** Bei Supabase die Log-Aufbewahrung des Tarifs prüfen und dokumentieren (API- und Edge-Logs enthalten IP-Adressen). Edge Functions loggen keine Request-Bodies und keine Header. Region EU, Auftragsverarbeitungsvertrag abschließen. Ab Wachstum prüfen: selbst gehostetes Supabase oder eigener schlanker Proxy mit Logformat ohne IP.
  3. **Fehlkonfiguration abfangen:** RLS-Test in der CI (ein Test-Client mit Anon-Key versucht `select` auf alle Tabellen und muss scheitern). Löschjob per `pg_cron` plus Healthcheck: Gibt es Zeilen älter als die Frist, geht ein Alarm an den Betreiber. Besser noch: Tabelle pro Tag partitionieren und alte Partitionen per `DROP` entfernen.
  4. **Push und Telegram aus dem MVP heraushalten** (siehe M-10, M-11). Falls Push kommt: Themen-Abos pro Linie und Richtung über FCM-Topics, der Betreiber speichert keine Tokens und keine Zuordnung Token ↔ Konto. Falls ein Telegram-Bot kommt: nur Buttons, kein Freitext, die Telegram-User-ID wird mit dem Tagesschlüssel gehasht und nicht gespeichert, Gewicht wie Stufe W.
  5. **Organisatorisch:** Verein statt Privatperson (KONZEPT 8/11.2), Anwalt vor dem Launch, ein kurzer Ablaufplan für Anfragen (wer antwortet, was wir haben, was nicht), jährlicher Transparenzbericht.
- **Status:** 🟡 teilweise
- **Restrisiko:** Hoster-Logs mit IP für die Dauer der Aufbewahrung, Metadaten bei Apple/Google (Attestierung, Push) und Telegram lassen sich nicht abschalten. Eine Anordnung, künftig mehr zu speichern, kann der Betreiber nicht technisch verhindern. Zivilrechtliche Ansprüche gegen den Betreiber (Unterlassung) bleiben ein offenes Risiko, unabhängig von der Technik. Transparenz: Datenschutzerklärung nennt Hoster, Log-Dauer und Drittanbieter ehrlich.

### M-08 Konkurrenz sabotiert
- **Lösung:**
  1. **Fake-Meldungen:** wie M-03. Für den Launch zusätzlich eine „Beta“-Kennzeichnung und die Notbremse mit niedrigerer Schwelle (3-faches Mittel), weil anfangs fast alle Konten S0 sind.
  2. **Kaltstart-Regel:** In den ersten 4 Wochen reicht für 🟠 eine einzelne Meldung (wie immer). 🔴 bleibt an unabhängige Cluster gebunden. So wirkt die App nicht leer, Fakes erzeugen aber kein „bestätigt“.
  3. **Store-Angriffsfläche verkleinern:** Repo „schwarzfahr“ vor dem Launch umbenennen oder privat stellen; neutraler App-Name; Store-Text beschreibt wahrheitsgemäß Abfahrten, Zugpositionen **und** Fahrgastmeldungen zu Kontrollen, ohne Aufforderung zum Fahren ohne Ticket (KONZEPT 8). In der App ein sichtbarer Hinweis „Bitte immer mit gültigem Ticket fahren“.
  4. **Web-Version ab Tag 1** (KONZEPT 7), damit eine Store-Entfernung die Nutzer nicht verliert.
  5. **Bewertungen:** öffentlich und sachlich antworten, offensichtliche Kampagnen über die Meldewege der Stores anzeigen.
- **Status:** 🟡 teilweise
- **Restrisiko:** Store-Entscheidungen und Bewertungen liegen nicht in der Hand des Betreibers (FreiFahren 2026, Akha 2025). Ein Store-Ausschluss bleibt wahrscheinlich genug, dass er eingeplant werden muss. Transparenz: Statusseite bzw. Hinweis in der Web-App, falls die Store-Version entfernt wurde.

### M-09 Spaßmeldungen von Schülergruppen
- **Lösung:**
  1. **Cluster-Deckel:** Zehn Meldungen derselben Fahrt, derselben Station, innerhalb von 3 Minuten zählen zusammen höchstens 1,5. Das reicht für 🟠, nicht für 🔴.
  2. **Ein Beitrag pro Konto und Ereignis:** Gegenseitiges Bestätigen innerhalb der Gruppe erhöht nichts.
  3. **Kein Vertrauensgewinn durch die eigene Gruppe** (M-04, Punkt 3). Junge Konten bleiben S0.
  4. **Nachträgliche Bewertung:** Endet ein Ereignis ohne jede unabhängige Bestätigung und ohne Ausstiegsmeldung, wird es für alle beteiligten Konten als „unbestätigt“ gezählt. Ab 3 solchen Ereignissen in 7 Tagen bleibt ein Konto für 14 Tage auf S0.
  5. **Folge-Meldungen der Gruppe** (z. B. `AN_STATION` Harburg) sind ein neues Ereignis, das wieder nur denselben Cluster enthält, also ebenfalls nur 🟠.
- **Status:** 🟡 teilweise
- **Restrisiko:** Ein Zug kann für einige Minuten 🟠 sein, ohne dass jemand kontrolliert. Das lässt sich nicht verhindern, ohne echte Einzelmeldungen ebenfalls zu unterdrücken. Transparenz: 🟠 sagt „1 Gruppe von Meldern, nicht unabhängig bestätigt“ (in Worten, nicht nur als Farbe), damit niemand 🟠 wie 🔴 liest.

### M-10 Gefährliche Flucht und Gedränge
- **Lösung:**
  1. **Nie eine Benachrichtigung für den Zug, in dem man eingecheckt ist**, und nie für einen Zug, der in den nächsten 2 Minuten an der eigenen Station hält. Push (KONZEPT Phase 3) nur für Favoritenstrecken **vor** Fahrtantritt und nur mit Liniennamen („Meldungen auf der S1“), nicht pro Zug.
  2. **Ruhige Darstellung während der Fahrt:** Statuswechsel im eingecheckten Zug ohne Ton, ohne Vibration, ohne Vollbild. Die Leiste (KONZEPT 4.5) wechselt nur die Farbe des kleinen Zugsymbols.
  3. **Keine Handlungsaufforderung im Wording:** nicht „Achtung! Kontrolleure! Jetzt aussteigen“, sondern „Kontrolle gemeldet · 1 Meldung · vor 2 Min.“. Keine Countdowns bis zur Tür, keine Hinweise auf Ausgänge.
  4. **Sicherheitshinweis** einmal im Onboarding und in den FAQ: „Steig nie hektisch aus und renn nicht am Bahnsteig. Meldungen können falsch oder dem falschen Zug zugeordnet sein.“
  5. **Zuordnungsfehler senken** (M-12, M-13), damit weniger Fehlalarme entstehen.
- **Status:** 🟡 teilweise
- **Restrisiko:** Wer die App im Zug offen hat und Rot sieht, kann trotzdem panisch reagieren. Das Verhalten der Menschen kann die App nicht steuern, nur nicht anheizen. Der Verzicht auf Pushs in den eigenen Zug kostet Nutzen, weil man nicht mehr aktiv gewarnt wird. Das ist bewusst so gewählt. Transparenz: Die FAQ erklärt, warum es diese Pushs nicht gibt.

### M-11 Belästigung von Kontrolleuren
- **Lösung:**
  1. **Kein offizieller Bot, der in Gruppen mit Freitext spiegelt.** Der Telegram-Plan aus KONZEPT 9.3 wird ersetzt durch: höchstens ein Kanal ohne Kommentarfunktion, der nur Liniensummen postet („S1 Ri. Wedel: Meldung aktiv“), ohne Zug, ohne Uhrzeit auf die Minute.
  2. **Keine offene Schnittstelle für Dritte:** Weil es nur den aggregierten Zustand gibt (M-05), können Spiegel-Bots keine Rohmeldungen, IDs oder Sekunden-Zeitpunkte abgreifen.
  3. **Keine Personendaten in der App:** kein Freitext, keine Fotos, keine Anzahl oder Beschreibung der Kontrolleure, keine Teamwege als Linie über mehrere Fahrten, keine Historie über das Ereignis hinaus (KONZEPT 4.4 bleibt, wird um „keine Teamverläufe“ ergänzt).
  4. **Statistiken nur grob** (M-05, Punkt 4), damit keine Dienstmuster einzelner Teams erkennbar werden („jeden Dienstag …“).
  5. **Nutzungsregeln und Abgrenzung:** Die Nutzungsbedingungen verbieten Fotos, Beschreibungen und Verfolgen. Der Betreiber distanziert sich öffentlich von Gruppen, die das tun, und meldet sie bei Telegram.
  6. **Unschärfe bewertet:** Eine Verzögerung von 10+ Minuten oder nur Linienebene in der App würde Verfolgung erschweren, nimmt der App aber ihren Kernnutzen (zugbezogene Warnung). Deshalb: in der App volle Genauigkeit auf Zug- und Stationsebene, aber nichts feiner (keine Wagen, keine Türen außer vorne/hinten für die Flügelung). In allen Spiegeln nach außen nur Linienebene.
- **Status:** 🟡 teilweise
- **Restrisiko:** Wer einem Team folgen will, sieht in der App „dieser Zug“. Das kann man nicht entfernen, ohne die App abzuschaffen. Was Dritte mit Screenshots in eigenen Gruppen tun, kann der Betreiber nicht kontrollieren. Transparenz: Regeln sind beim ersten Melden sichtbar, und die FAQ erklärt, warum es keinen Freitext gibt.

### M-12 Verspätungsdaten gefälscht, Meldungen landen im falschen Zug
- **Lösung:**
  1. **Server berechnet die Verspätung**, nicht jedes Handy für sich: Median aus ≥ 2 unabhängigen, attestierten Konten S1/S2 pro Fahrt (S0 und W werden ignoriert). Verteilt wird nur der Medianwert.
  2. **Plausibilitätsregeln:** Ein Konto liefert Verspätungen nur für die eine Fahrt, in der es aktiv ist. Werte müssen entlang der Fahrt stetig sein (max. ±2 Min. Änderung pro Abschnitt). Ein Zug kann seinen Vorgänger auf derselben Strecke nicht überholen. Verspätungen über 20 Min. gelten erst ab 3 Quellen.
  3. **Ohne Check-in keine stille Zuordnung:** Die Meldung eines nicht eingecheckten Nutzers geht als „Linie, Richtung, Station, Zeitpunkt“ an den Server. Der Server ordnet sie der Fahrt zu, die laut bester Schätzung dort ist. Ist das nicht eindeutig (zwei Fahrten innerhalb von ±3 Min.), werden **beide** Fahrten 🟠 statt einer falsch 🔴 und einer ⚪. Die App fragt vorher mit einem Tap: „Zug steht gerade in Altona?“.
  4. **Offizielle Daten als Anker** (KONZEPT 6.4 Stufe 3): Sobald DB-Timetables-Daten vorliegen, gewinnen sie. Starke Abweichungen der Crowd-Werte davon setzen die Crowd-Werte dieser Fahrt auf 0.
  5. **Anzeige:** „+3 Min. (Fahrgastangabe)“ statt einer stillen Verschiebung.
- **Status:** ✅ gelöst
- **Restrisiko:** Mehrere echte, etablierte Konten in derselben Fahrt könnten eine falsche Verspätung durchsetzen. Ohne offizielle Daten bleibt die Position eine Schätzung. Transparenz: Kennzeichnung „Fahrgastangabe“ und „Fahrplan“ an jeder Zeit.

### M-13 Verwechslung bei der Flügelung in Ohlsdorf
- **Lösung:**
  1. **Zugteil aus dem Check-in ableiten:** Wer in die Airport-Fahrt eincheckt, sitzt (wenn er richtig eingestiegen ist) im Airport-Teil. Die manuelle Angabe „vorne/hinten“ entfällt für Eingecheckte. Die App erinnert beim Check-in vor Ohlsdorf: „Airport = vorderer Teil“ (Wert aus den gepflegten Daten).
  2. **Asymmetrie auch hier:** Die genannte Meldung geht an den angegebenen Teil. Der andere Teil wird ab Ohlsdorf für 10 Minuten 🟠 „evtl. im anderen Zugteil“ und nicht ⚪. Eine falsche Angabe erzeugt so nie eine falsche Sicherheit.
  3. **Korrektur durch spätere Meldungen:** Eine Meldung aus einem Teil nach Ohlsdorf setzt diesen Teil auf mindestens 🟠 bzw. 🔴, unabhängig von der früheren Angabe.
  4. **Ein Einzelner entscheidet nichts:** Eine Zugteil-Angabe zählt nur mit Gewicht, wenn das Konto mindestens S1 ist. Sonst erben beide Teile die Meldung als 🟠 (bereits KONZEPT 5.4).
- **Status:** ✅ gelöst
- **Restrisiko:** Wer in den falschen Zugteil eingestiegen ist, meldet für die falsche Fahrt. Der 🟠-Puffer auf dem anderen Teil fängt das für die ersten Minuten nach Ohlsdorf ab. Transparenz: Text „gemeldet vor der Zugteilung in Ohlsdorf“ auf beiden Ästen.

---

## Querschnitt

Die wichtigsten Änderungen am Konzept, die mehrere Szenarien gleichzeitig lösen:

1. **Asymmetrische Zustandsmaschine** (KONZEPT 5.1, 5.2, 5.3 Nr. 1 und 5, 5.4, 11.4): Warnungen werden leicht sichtbar (🟠 ab einer attestierten Meldung), Entwarnungen wirken schwer und nie sofort. `AUSGESTIEGEN_IN` macht aus 🔴 höchstens 🟠, ⚪ nur durch Fahrtende, Zeitablauf oder mehrere unabhängige etablierte Entwarnungen plus 10 ruhige Minuten. Jede neue Warnung gewinnt. 🔴 nur aus ≥ 2 unabhängigen Clustern. → M-01, M-02, M-04, M-09, M-13.

2. **Server ist die Instanz, die den Zustand berechnet, Rohmeldungen bleiben intern** (KONZEPT 7 „Architektur“ und „Datenmodell“, dort „Status-Berechnung lokal“ und „jedes Handy lädt alle Meldungen“ ersetzen): RLS sperrt das Lesen von `meldungen`, Schreiben nur über eine Edge Function mit Plausibilitätsprüfung gegen den Fahrplan, ausgeliefert wird nur der aggregierte Zustand (gerundete Zeiten, Anzahl in Stufen, keine IDs, keine Historie) per CDN-Snapshot und Realtime-Broadcast, mit 10–40 s zufälliger Verzögerung. Zugpositionen bleiben auf dem Handy. → M-03, M-05, M-06, M-07, M-11, M-12.

3. **Attestierte, entkoppelte Konten statt freier Geräte-ID** (KONZEPT 5.3 Nr. 2–4, Datenmodell-Spalte `geraet`, 10 Phase 1 „Anonyme Geräte-ID“): App Attest/Play Integrity zum Schreiben, DeviceCheck/Device Recall gegen Neuinstallations-Tricks, Stufen nach Alter und **fremd** bestätigten Meldungen, Gewichte mit Deckel, Konto-Bezug einer Meldung nur bis Ereignisende. Web und de-googelte Geräte lesen voll und warnen mit geringem Gewicht. → M-01, M-03, M-04, M-06, M-08, M-09, M-12.

4. **Datenminimierung über die eigenen Tabellen hinaus** (KONZEPT 7 „Kein … keine IP-Speicherung“, Löschjob, 8, 10 Phase 3 Push und Telegram-Bot, 9.3): Rohmeldungen nur bis Ereignisende + 2 h, Partitionen per `DROP`, RLS-Test und Löschjob-Alarm in der CI, Hoster-Log-Aufbewahrung dokumentieren und minimieren, Push nur als Linien-Topic ohne gespeicherte Tokens, Telegram nur als Kanal mit Liniensummen und ohne Freitext-Eingang. Datenschutzerklärung sagt offen, was Mitlesende speichern können. → M-05, M-06, M-07, M-11.

5. **Sicherheits- und Wording-Regeln als feste Produktregel** (KONZEPT 4.1, 4.5, 5.2, 8, 10 Phase 3 Push): keine Benachrichtigung für den eigenen oder gleich einfahrenden Zug, ruhige Darstellung im Check-in, keine Handlungsaufforderungen oder Ausweichrouten, ⚪ = „keine Meldung, Kontrollen können überall sein“, 🟠/🔴 mit Text statt nur Farbe, Hinweis „mit gültigem Ticket fahren“, keine Teamverläufe, nur grobe Statistik. → M-02, M-08, M-10, M-11.

6. **Neuer Abschnitt „Notbremse und Transparenz“** (Ergänzung zu KONZEPT 5.3): Bei ungewöhnlich vielen Meldungen (5-faches Mittel, beim Launch 3-fach) oder gleichzeitigen Entwarnungen vieler bewährter Konten gelten für eine begrenzte Zeit strengere Regeln, und die App sagt das sichtbar. → M-02, M-03, M-04, M-08.
