# Szenarien Z: Zugzuordnung

### Z-01 Verspätung im 5-Minuten-Takt: Meldung landet einen Zug zu früh
- Ablauf: HVZ, S1 Richtung Ohlsdorf, Züge im 5-Minuten-Takt. Der Zug „S1|Wedel|08:12“ hat 6 min Verspätung. A steigt um 08:45 in Altona ein, tippt im Fahrplan „nächster Zug“ an und erwischt damit „08:17 ab Wedel“. Das ist aber der Zug, der planmäßig um 08:44 kommt und selbst 4 min zu spät ist. A sieht in Altona F1 und F2 einsteigen und meldet „im Zug“.
- App vs. Wirklichkeit: Rot ist der Zug 08:17. F1/F2 sitzen aber im Zug 08:12, der eine Lücke von 1 min zum Folgezug hat. Am Hbf sieht B „08:12 grün“ und steigt genau in diesen Zug.
- Folge: B wird kontrolliert. Wer im echten 08:17 sitzt, hat ohne Grund Angst. Bei engem Takt reicht schon eine Verspätung von mehr als 2,5 min, damit der Zug falsch zugeordnet wird.
- Schwere: hoch

### Z-02 Zugausfall: Meldung hängt an einem Geisterzug
- Ablauf: Die S3 „Pinneberg 09:05“ fällt aus, aber die App kennt keine Echtzeitdaten. Der nächste echte Zug ist 09:15 und deutlich voller. C ist um 09:30 in Eidelstedt eingecheckt. Weil 09:05 als „der nächste“ angezeigt wurde, ist C in diesen Ausfall-Zug eingecheckt und meldet um 09:35 in Diebsteich F1.
- App vs. Wirklichkeit: Der Ausfall-Zug fährt laut App mit roter Meldung weiter und „erreicht“ schon um 09:50 Neugraben. Der echte 09:15 bleibt grün.
- Folge: Alle zwischen Altona und Harburg vertrauen dem grünen 09:15. Die Meldung „verschwindet“ am Endbahnhof, bevor der echte Zug dort ist.
- Schwere: hoch

### Z-03 Kurzwende in Barmbek: Meldung fährt in der App weiter nach Ohlsdorf
- Ablauf: Wegen einer Störung wendet die S1 „Wedel 10:12“ in Barmbek und fährt als Gegenzug Richtung Wedel zurück. F1 bleibt im Zug. D meldet um 11:10 in Barmbek „im Zug“.
- App vs. Wirklichkeit: Laut App ist der Zug um 11:20 am Airport. In Wirklichkeit fährt er mit F1 Richtung Hasselbrook und Hbf, unter einer Fahrt-ID, die die App gar nicht kennt.
- Folge: E steigt um 11:18 in Wandsbeker Chaussee in diesen Zug Richtung Wedel. Dort ist keine Meldung, gemeldet ist er nur am Airport. Am Flughafen gibt es Fehlalarm.
- Schwere: mittel bis hoch

### Z-04 Flügelung Ohlsdorf: Kontrolleur geht in den falschen Zugteil
- Ablauf: Die Meldung „S1|Wedel|08:12“ (physischer Zug, ID der Airport-Fahrt) kommt um 09:12 nach Ohlsdorf. F1 und F2 sind im hinteren Teil. Kurz vor der Trennung geht F2 über den Bahnsteig in den vorderen Teil.
- App vs. Wirklichkeit: Die Meldung gilt für die Airport-Fahrt, die Poppenbüttel-Fahrt verweist nur per `partnerOf` darauf. Die App muss nach der Trennung raten. Wenn sie die Meldung nur am Airport-Teil weiterführt, ist der Poppenbüttel-Teil mit F1 grün. Wenn sie beide markiert, ist das nur zufällig richtig, weil F2 gewechselt hat.
- Folge: G steigt um 09:16 in Kornweg in den „grünen“ Poppenbüttel-Teil. Ohne Angabe des Zugteils ist die Meldung ab Ohlsdorf grundsätzlich falsch oder unscharf.
- Schwere: hoch

### Z-05 Vereinigung in Ohlsdorf (Gegenrichtung): Meldung geht verloren
- Ablauf: H meldet um 14:03 in Hoheneichen (Poppenbüttel-Teil) F1. In Ohlsdorf werden die Teile vereinigt, Weiterfahrt als Zug „Airport 14:xx“ Richtung Wedel.
- App vs. Wirklichkeit: Die Meldung hängt an der Poppenbüttel-Fahrt. Wenn die Vereinigung nicht modelliert ist, endet sie in Ohlsdorf. Der vereinigte Zug ab Barmbek ist grün.
- Folge: Wer zwischen Barmbek und Hbf einsteigt, wird nicht gewarnt.
- Schwere: mittel

### Z-06 HVZ-Verstärker ohne Flügelung wird als Flügelzug gezeigt
- Ablauf: Der Verstärker Blankenese–Poppenbüttel um 07:48 fährt fast zeitgleich mit einem Wedel-Vollzug durch Altona (2 min Abstand). J checkt im Tunnel ohne GPS ein und wählt aus Versehen den Wedel-Zug. Er meldet F1.
- App vs. Wirklichkeit: Die Meldung erscheint am Airport-Teil, F1 fährt aber nach Poppenbüttel.
- Folge: Wellingsbüttel und Poppenbüttel sind „grün“. Am Flughafen gibt es Fehlalarm.
- Schwere: mittel

### Z-07 S5 in Neugraben: Meldung bricht am GTFS-Fahrtende ab
- Ablauf: K meldet um 16:40 in Harburg F1 und F2 in der S5 „Elbgaustraße 16:05“. Im GTFS endet diese Fahrt in Neugraben. Der Zug fährt als zweite Fahrt nach Stade weiter (bei einem kleinen Teil der Fahrten ist nicht klar, welche Fahrt dazugehört).
- App vs. Wirklichkeit: Ab Neugraben ist der Zug laut App ein neuer, grüner Zug. F1 und F2 kontrollieren bis Buxtehude.
- Folge: L in Fischbek und Neu Wulmstorf wird nicht gewarnt. Wenn die Fahrten falsch zusammengefügt sind, bekommt ein anderer Zug die Meldung, etwa die S3 nach Neugraben, die dort endet.
- Schwere: mittel bis hoch

### Z-08 S7-Schleife: Richtung und Abschnitt wechseln mitten in der Fahrt
- Ablauf: Die S7 fährt Bergedorf → City-Tunnel → Altona → Verbindungsbahn → Bergedorf. M meldet um 12:20 in Jungfernstieg F1 in „S7 Richtung Altona“. F1 fährt in Altona mit weiter Richtung Holstenstraße und Dammtor.
- App vs. Wirklichkeit: Wenn die App die Schleife als zwei Fahrten oder zwei „Richtungen“ führt, endet die Meldung in Altona. Auf dem Streifen „S7 Richtung Hbf“ ist der Zug grün. Außerdem steht die S7 in der Abfahrtstafel Dammtor neben S2 und S5. Ein Nutzer kann dort einen S5-Zug für den S7-Zug halten.
- Folge: N wird in Sternschanze kontrolliert.
- Schwere: mittel

### Z-09 City-Tunnel: drei Linien, falsche Linie gemeldet
- Ablauf: Zwischen Hbf und Altona fahren S1, S3 und S7 alle 2 bis 3 min. P sitzt um 17:31 in Landungsbrücken in der S3, verwechselt sie mit der S1 und checkt „S1 17:30“ ein. P meldet F1.
- App vs. Wirklichkeit: Die S1 ist rot, die S3 grün. Ab Altona fährt die S1 nach Wedel, die S3 nach Pinneberg.
- Folge: Q in Diebsteich und Stellingen (S3) wird nicht gewarnt. In Bahrenfeld wird ohne Grund gewarnt. Weil die Abschnitte gemeinsam sind, wird der Fehler erst nach der Verzweigung sichtbar.
- Schwere: hoch

### Z-10 Zwei Züge gleichzeitig am Bahnsteig (Hbf / Ohlsdorf)
- Ablauf: Am Hbf stehen um 08:02 eine S3 nach Neugraben und eine S5 nach Stade gleichzeitig an gegenüberliegenden Gleisen. R sieht F1 und F2 einsteigen und meldet „im nächsten Zug Richtung Harburg“. Die App wählt die S3, sie fahren aber in der S5.
- App vs. Wirklichkeit: Bis Harburg sind beide Linien auf derselben Strecke, also stimmt die Warnung grob. Ab Harburg ist die falsche Linie rot. Neuwiedenthal, Fischbek und Buxtehude stimmen nicht.
- Folge: S in Fischbek (nur S5) steigt arglos ein.
- Schwere: mittel

### Z-11 GPS im Tunnel: Station falsch zugeordnet
- Ablauf: T steht im City-Tunnel am Bahnsteig Stadthausbrücke. Das GPS hat den letzten Fix von Jungfernstieg oder springt über den Tunnel-Mittelpunkt. T meldet per GPS „Kontrolleure am Bahnsteig“.
- App vs. Wirklichkeit: Die Stationsmeldung landet bei Jungfernstieg. Diese Station ist am meisten befahren und gilt für alle Linien. Stadthausbrücke bleibt grün.
- Folge: U steigt in Stadthausbrücke direkt bei F1 aus. Viele andere meiden Jungfernstieg ohne Grund, die Glaubwürdigkeit leidet.
- Schwere: mittel

### Z-12 Vergessener Check-out: Ein Geistermelder bestätigt falsche Züge
- Ablauf: V checkt um 07:40 in die S1 ein und steigt in Hasselbrook aus, ohne auszuchecken. Um 08:15 sitzt V in der U-Bahn. Die App fragt nach „Kontrolleure noch im Zug?“, und V tippt nebenbei „ja“.
- App vs. Wirklichkeit: Die Meldung am S1-Zug wird wieder bestätigt, obwohl F1 schon in Barmbek ausgestiegen ist. Oder V's Handy liefert eine Position (U-Bahn) als „Zugposition“.
- Folge: Ein Zug, der längst kontrollfrei ist, bleibt bis zum Airport rot. Die Warnungen verlieren an Wert.
- Schwere: niedrig bis mittel

### Z-13 Ersatzverkehr wegen Bauarbeiten: Die App zeigt Züge, die nicht fahren
- Ablauf: Am Wochenende gibt es zwischen Harburg und Neugraben Schienenersatzverkehr. Die App rechnet mit dem Regel-GTFS. W fährt im Ersatzbus und checkt in „S3 10:12“ ein, weil kein Zug angeboten wird. W meldet in Heimfeld F1 und F2 im Bus.
- App vs. Wirklichkeit: Ein Zug, der nicht fährt, wird rot und fährt virtuell nach Neugraben. Der Bus wird nicht dargestellt. Die Stationsmeldung „Heimfeld“ gilt außerdem für den S-Bahnsteig, nicht für die Bushaltestelle.
- Folge: Nutzer verlassen sich auf eine Darstellung, die gar nicht existiert. Busfahrgäste werden nicht gewarnt.
- Schwere: mittel

### Z-14 Fahrplanwechsel Dezember 2026: S7 entfällt, S2 fährt wieder nach Altona
- Ablauf: Am 13.12.2026 stellt die S-Bahn um. Das monatliche JSON stammt noch vom 1.12. In Bergedorf meldet X am 14.12. um 07:50 F1 in „S7 07:48“. Der Zug ist in Wirklichkeit eine S2 nach Altona über den Hbf. Dazu kommt, dass der `fahrtKey` (Linie, Start, Datum, Abfahrt) bei Handys mit neuem und altem Datensatz unterschiedlich ist.
- App vs. Wirklichkeit: Handys mit altem Datensatz zeigen die S7-Schleife. Handys mit neuem Datensatz sehen die Meldung nicht, weil ihr Schlüssel nicht passt. Die Meldung ist auf zwei Datenwelten aufgeteilt.
- Folge: Mehrere Tage gibt es systematisch falsche oder verlorene Meldungen auf dem ganzen Bergedorfer Ast.
- Schwere: hoch

### Z-15 Kontrolleure steigen um, Meldung bleibt am alten Zug
- Ablauf: F1 und F2 kontrollieren die S1 von Blankenese bis Altona und steigen dort in die S3 nach Pinneberg um. Niemand meldet „ausgestiegen“. Y meldet 4 min später in Diebsteich neu, und die App fasst das nicht als dieselben Personen zusammen.
- App vs. Wirklichkeit: Die S1 ist bis Ohlsdorf rot, obwohl sie leer ist. Die S3 ist erst ab Diebsteich rot. In Altona gibt es keine Stationsmeldung, obwohl die Kontrolleure dort am Bahnsteig standen.
- Folge: Die S1-Fahrgäste haben unnötig Angst. Das Zeitfenster für Fahrgäste in der S3 zwischen Altona und Diebsteich ist ohne Warnung. Es gibt doppelt so viele Meldungen, als Kontrolleure unterwegs sind.
- Schwere: mittel
