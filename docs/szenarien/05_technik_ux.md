# Szenarien T: Technik, Nutzerdichte und Bedienung

### T-01 Tunnel-Meldung kommt zu spät (City-S-Bahn)
- Ablauf: A sitzt 08:10 in der S1 Richtung Poppenbüttel. Am Hbf steigen Kontrolleure zu, A meldet zwischen Hbf und Jungfernstieg im Tunnel, ohne Netz. Die App hält die Meldung zurück und schickt sie erst ab, als in Landungsbrücken (08:16) wieder Netz da ist. B steht am Jungfernstieg und steigt um 08:14 in diesen Zug.
- App vs. Wirklichkeit: B sieht bis 08:16 „⚪ keine Meldung“, obwohl seit 08:12 kontrolliert wird. Wenn die Meldung mit dem Zeitstempel vom Versand statt vom Tippen gespeichert wird, wirkt sie außerdem 4 min frischer, als sie ist, oder sie hängt an der falschen Station.
- Folge: B wird kontrolliert und gibt der App die Schuld. Für die Meldungen im Tunnelabschnitt sinkt das Vertrauen.
- Schwere: hoch

### T-02 Außenast früh morgens: keine Meldung heißt „niemand da“, nicht „sicher“
- Ablauf: 05:20, S5 ab Stade. Auf der ganzen Strecke bis Neugraben ist nur C mit der App unterwegs, im hinteren Wagen. Die Kontrolleure gehen vorne durch. Wedel um 23:40 sieht genauso aus.
- App vs. Wirklichkeit: Der ganze Streifen ist grau/⚪. Die App unterscheidet nicht zwischen „geprüft, nichts los“ und „niemand hat hingeschaut“.
- Folge: D im vorderen Wagen fühlt sich sicher und wird erwischt. Für Pendler aus Stade oder Pinneberg ist die App fast wertlos. Sie hören auf, sie zu nutzen, und so bleibt die Strecke dauerhaft ohne Meldungen.
- Schwere: hoch

### T-03 Push kommt zu spät (Doze-Modus, iOS-Drosselung)
- Ablauf: E meldet 17:02 Kontrolleure in der S3 an der Sternschanze. F hat Altona als Favorit und Push an, das Android-Handy liegt aber im Akku-Sparmodus. Der Push kommt 17:11 an, da sitzt F schon in genau diesem Zug.
- App vs. Wirklichkeit: Die Meldung ist auf dem Server sofort da, aber auf F's Sperrbildschirm erst 9 min später, und dort wird „Sternschanze“ angezeigt, obwohl die Kontrolleure längst weitergefahren sind.
- Folge: F wird kontrolliert, obwohl die App „rechtzeitig“ Bescheid wusste. Das Push-Versprechen ist nichts wert.
- Schwere: mittel

### T-04 Bewegungserkennung meldet „ausgestiegen“, obwohl der Nutzer noch im Zug ist
- Ablauf: G hat den Zug an der S1 in Ohlsdorf als „mit Kontrolleuren“ markiert und „Ich bin in diesem Zug“ gedrückt. Weil der Zug lange am Bahnsteig steht und die Flügelung dauert, läuft G nach vorne in den Airport-Teil. Die Bewegungserkennung deutet Gehen plus Stillstand als „ausgestiegen“. Umgekehrt steht H in Barmbek am Bahnsteig, läuft hin und her, und die App glaubt, H sei noch im Zug.
- App vs. Wirklichkeit: G's Zug zeigt keinen Augenzeugen mehr, die Meldung läuft schneller ab. Die Meldung von H wird an eine Fahrt gehängt, in der H gar nicht sitzt.
- Folge: Die Zugmeldung ist falsch oder geht verloren. Bei der Flügelung landet die Meldung schnell im falschen Zugteil (Airport oder Poppenbüttel).
- Schwere: mittel

### T-05 Falsche Handy-Uhrzeit verschiebt alle Züge
- Ablauf: Bei I ist die automatische Zeit aus, das Handy geht 6 min nach (oder steht nach der Zeitumstellung Ende Oktober eine Stunde falsch). I meldet um 09:30 echter Zeit „im Zug“ an der Station Holstenstraße.
- App vs. Wirklichkeit: Die Position wird lokal aus dem GTFS berechnet, also zeigt I's App einen anderen Zug an der Station als den echten. Die Meldung bekommt den fahrtKey des Zugs davor oder danach, z. B. „S3|…|09:24“.
- Folge: Der falsche Zug wird rot und der richtige bleibt ⚪. Alle anderen werden in die Irre geführt, und I merkt davon nichts.
- Schwere: hoch

### T-06 Supabase-Ausfall oder Free-Tier-Limit erreicht
- Ablauf: An einem Freitagabend geht ein Post in den sozialen Medien viral, die Zahl der Nutzer steigt stark, die Realtime-Verbindungen des kostenlosen Tarifs sind voll. Ab 22:00 nimmt der Server keine Meldungen mehr an, die App fängt den Fehler aber still ab.
- App vs. Wirklichkeit: Überall steht „⚪ keine Meldung“. Es gibt keinen Hinweis wie „Server nicht erreichbar, Daten von 21:58“.
- Folge: Alle Nutzer glauben an ein ruhiges Netz, und die Meldungen der Melder gehen verloren. Das ist schlimmer als eine App, die gar nicht startet.
- Schwere: hoch

### T-07 Gecachte Meldungen von gestern
- Ablauf: J öffnet die App morgens um 07:50 im Funkloch am Bahnsteig Hammerbrook. Die App zeigt den letzten Stand von gestern 18:30 aus dem Cache: „🔴 Station Hammerbrook, Bahnsteig“. Der fahrtKey enthält zwar das Datum, die Stationsmeldung hat aber nur eine Uhrzeit, kein Datum.
- App vs. Wirklichkeit: J sieht eine aktuelle Kontrolle, die es nicht gibt, wartet auf den nächsten Zug oder nimmt den Bus. Bei anderen Nutzern läuft das in die andere Richtung: „Alles grün“ aus dem Cache, obwohl gerade kontrolliert wird.
- Folge: J verliert Zeit und das Vertrauen in die App. Wenn auf dem Bildschirm kein Hinweis „Stand: gestern“ steht, merkt man es nicht.
- Schwere: mittel

### T-08 Meldung aus Versehen in der Hosentasche
- Ablauf: K hat die Abfahrtstafel offen und steckt das Handy weg. Der große Button „Kontrolle melden“ wird in der Tasche ausgelöst, die App nimmt per GPS Station Dammtor und den nächsten Zug. Das passiert um 16:40 in der Rushhour.
- App vs. Wirklichkeit: S21 und S31 werden an der Station Dammtor rot angezeigt, es kontrolliert aber niemand. Weil kaum andere Nutzer unterwegs sind, widerspricht keiner, und die Meldung bleibt lange stehen.
- Folge: Viele Nutzer weichen unnötig aus. Häufen sich solche Fälle, wirkt Rot beliebig und wird ignoriert, auch wenn die Meldung echt ist.
- Schwere: mittel

### T-09 Verlässt sich zu sehr auf die App (Automatisierungs-Vertrauen)
- Ablauf: L fährt seit Wochen ohne Ticket von Bergedorf nach Hbf und schaut nur auf die App. Seit Wochen war jede Fahrt ⚪, also verlässt sich L darauf. Heute gibt es einen Schwerpunkt-Einsatz mit 8 Kontrolleuren in Zivil. Sie steigen gleichzeitig an allen Türen zu, und keiner kommt mehr zum Melden.
- App vs. Wirklichkeit: Der Zug ist ⚪, obwohl gerade alle kontrolliert werden. Gerade bei großen Einsätzen fehlen die Meldungen.
- Folge: L muss 60 € erhöhtes Beförderungsentgelt zahlen, bei Wiederholung droht eine Anzeige. Der Schaden trifft den Nutzer direkt, und für den Betreiber kommt der Vorwurf dazu, er stifte zum Schwarzfahren an.
- Schwere: hoch

### T-10 Mehrere Geräte und doppelte Bestätigungen
- Ablauf: M hat Handy und Tablet mit demselben Konto (oder zwei anonymen IDs) dabei und meldet bzw. bestätigt in Eidelstedt auf beiden Geräten. N nutzt bewusst drei Geräte, um mit Fake-Meldungen Strecken „freizuhalten“ oder „zu sperren“.
- App vs. Wirklichkeit: Die Meldung wird als „3 Bestätigungen, sicher“ angezeigt, obwohl nur eine Person dahintersteht. Die Unsicherheit (🟠) lässt sich so leicht wegdrücken.
- Folge: Die Vertrauensstufen sind nichts mehr wert, und es kann gezielte Manipulation geben, etwa durch Betreiber-Personal oder Trolle.
- Schwere: mittel

### T-11 Datenschutzsorge wegen Bewegungs-Tracking
- Ablauf: O liest im Store „nutzt Standort im Hintergrund und Bewegungssensoren“. Ein Journalist fragt: Kann der Server sehen, wer täglich von Blankenese zur Hochschule fährt und dort ohne Ticket fährt? Die Meldungen sind mit Gerät-ID, GPS und Zeit gespeichert.
- App vs. Wirklichkeit: Die App sagt „anonym“, aus den Meldungen lassen sich aber Bewegungsprofile und Wohnorte ablesen. Bei einer Herausgabe an Behörden oder einem Datenleck ließe sich zeigen, wer regelmäßig fährt, wenn kontrolliert wird.
- Folge: Viele erlauben keine Bewegungserkennung, und es gibt weniger Meldungen. Möglich sind DSGVO-Beschwerden und Rufschaden. Für die Nutzer ist es ein rechtliches Risiko.
- Schwere: hoch

### T-12 Farben und Symbole werden falsch verstanden (Barrierefreiheit)
- Ablauf: P ist rot-grün-farbenblind und kann 🔴 und 🟠 auf dem Streifen kaum unterscheiden. Q denkt, ⚪ heiße „geprüft, sauber“, und hält 🔴 an der Station Blankenese für „der Zug ist hier“. R nutzt VoiceOver, die Emojis werden als „roter Kreis“ vorgelesen und das Wischen auf der Zugkarte geht mit dem Screenreader nicht.
- App vs. Wirklichkeit: Es gibt für Zug und Station dieselbe Farbe für verschiedene Dinge, und die App erklärt nicht, dass „keine Meldung“ nicht „keine Kontrolle“ bedeutet.
- Folge: Die Nutzer entscheiden falsch, und es werden Menschen mit Einschränkung ausgeschlossen. Eventuell sind Vorgaben zur Barrierefreiheit (BFSG) betroffen.
- Schwere: mittel

### T-13 App wird aus dem Store entfernt / Akku leer
- Ablauf: Nach Beschwerden der Betreiber wird die App im Dezember aus dem Play Store und dem App Store entfernt. Bestehende Installationen laufen weiter, bekommen aber kein GTFS- oder Fahrplan-Update mehr. Ab dem Fahrplanwechsel am 13.12. rechnen sie mit dem alten Fahrplan. Dazu kommt S, dessen Akku am Abend bei 3 % ist. Die App mit GPS und Sensoren leert ihn ganz, und S kann am Ende weder ein Handyticket zeigen noch melden.
- App vs. Wirklichkeit: Die Züge stehen an falschen Positionen und die fahrtKeys passen nicht mehr zu denen neuer Geräte, also landen Meldungen ins Leere. S verschwindet ohne Hinweis als Melder.
- Folge: Das System zerfällt nach und nach, die Zahl der Nutzer geht gegen null. Für S kommt ein ungewollt hoher Akkuverbrauch dazu.
- Schwere: hoch (Store-Entfernung) / mittel (Akku)
