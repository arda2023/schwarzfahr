# Szenarien S: Veraltete und falsche Meldungen

### S-01 Melder steigt aus, Kontrolleur verschwindet unbemerkt
- Ablauf: A steigt um 08:12 in Wedel in die S1 ein. In Rissen (08:15) sieht A, wie Kontrolleur F1 einsteigt, und meldet ihn. A steigt in Blankenese (08:24) aus, F1 bleibt im Zug. In Hochkamp (08:26) steigt F1 unbemerkt aus. B steht in Klein Flottbek (08:28) und sieht den Zug rot.
- App vs. Wirklichkeit: Die App zeigt „🔴 im Zug“, bis jemand eine Entwarnung meldet. Der Zug ist in Wirklichkeit seit Hochkamp frei.
- Folge: B lässt den Zug fahren und verliert 10 Minuten. Das wiederholt sich an jeder folgenden Station.
- Schwere: mittel

### S-02 Falsch-negativ: Kontrolleure steigen still zu
- Ablauf: F1 und F2 steigen um 17:40 in Altona in die S1 Richtung Ohlsdorf ein. Keiner der Fahrgäste nutzt die App. A steigt in Hammerbrook… genauer am Hbf (17:50) ein, weil der Zug „⚪ keine Meldung“ zeigt. Am Berliner Tor wird A kontrolliert.
- App vs. Wirklichkeit: Die App zeigt keine Meldung, im Zug sind aber Kontrolleure. Für Nutzer klingt „keine Meldung“ wie „sicher“.
- Folge: A zahlt 60 € und verliert das Vertrauen in die App. Das ist der häufigste Fall.
- Schwere: hoch

### S-03 Flügelung Ohlsdorf: Meldung landet im falschen Zugteil
- Ablauf: A meldet um 09:00 in Barmbek F1 in der S1 (Vollzug). In Ohlsdorf (09:06) wird der Zug geteilt, F1 fährt im vorderen Teil Richtung Airport. Die Meldung hängt am physischen Zug mit der Airport-fahrtKey, die Poppenbüttel-Fahrt verweist per `partnerOf` darauf. Entweder erben beide Teile die Meldung, oder es ist unklar, wohin sie gehört. B in Wellingsbüttel sieht den Poppenbüttel-Zug rot.
- App vs. Wirklichkeit: Beide Zugteile oder der falsche sind rot. Nur ein Teil ist kontrolliert.
- Folge: Der Poppenbüttel-Ast wird ohne Grund gemieden. Steht die Meldung dagegen nur am anderen Teil, läuft der Airport-Ast blind in die Kontrolle (falsch-negativ).
- Schwere: hoch

### S-04 Umstieg der Kontrolleure am Knoten
- Ablauf: A meldet um 10:10 F1 und F2 in der S3 ab Harburg. Am Hbf (10:22) wechseln F1 und F2 in die S1 Richtung Wedel. Die S3 bleibt rot. In der S1 meldet niemand etwas.
- App vs. Wirklichkeit: Die S3 ist rot, aber leer. Die S1 ist ohne Meldung, aber kontrolliert. Falsch-positiv und falsch-negativ treten gleichzeitig auf.
- Folge: Ausweichende Fahrgäste steigen gezielt in die S1 um und laufen F1 und F2 direkt in die Arme.
- Schwere: hoch

### S-05 Verfall nach Zeit statt nach Ereignis
- Ablauf: Angenommen, die Meldung läuft nach 20 Minuten ab. A meldet um 07:30 F1 in der S3 Pinneberg→Stade, F1 kontrolliert bis Neugraben. Nach 20 Minuten springt der Zug bei Wilhelmsburg auf ⚪ oder 🟠. C steigt in Harburg ein.
- App vs. Wirklichkeit: Die App zeigt „keine Meldung/unsicher“, obwohl F1 noch kontrolliert. Ist die Ablaufzeit länger, gilt das Gegenteil: S-01 tritt verstärkt auf.
- Folge: Jede feste Ablaufzeit ist bei langen Linien wie der S3 falsch, entweder zu früh oder zu spät.
- Schwere: mittel

### S-06 Stationsmeldung bleibt hängen
- Ablauf: A meldet um 16:00 eine Bahnsteigkontrolle am Jungfernstieg. Um 16:05 ziehen F1 bis F3 weiter zur Stadthausbrücke. Bis 16:30 meldet niemand eine Entwarnung. B fährt deshalb einen Umweg über Hbf/U-Bahn.
- App vs. Wirklichkeit: Die App zeigt „Jungfernstieg: Bahnsteig, 25 min“. Dort ist niemand, Stadthausbrücke ist ohne Meldung kontrolliert.
- Folge: Weil die Meldung für alle Linien gilt, sind S1, S2 und S3 betroffen. Der Fehler wirkt breit.
- Schwere: mittel

### S-07 Verspätung verschiebt die berechnete Zugposition
- Ablauf: Die S1, Abfahrt 08:12 ab Wedel, hat 8 Minuten Verspätung. A meldet aus dem Zug in Othmarschen. Die App ordnet die Meldung über die Fahrplanposition der planmäßigen 08:22-Fahrt zu, die physisch schon in Altona ist. B in Bahrenfeld sieht den falschen Zug rot.
- App vs. Wirklichkeit: Die Meldung hängt am Vorderzug. Der tatsächlich kontrollierte Zug ist ⚪.
- Folge: B meidet den leeren Zug und steigt in den kontrollierten ein.
- Schwere: hoch

### S-08 Kontrolleure fahren gegenläufig zurück
- Ablauf: F1 kontrolliert die S5 (über Buxtehude) Richtung Stade und wird gemeldet. In Neugraben steigt F1 aus und nimmt die Gegenrichtung zurück Richtung Harburg. Die Meldung fährt mit dem alten Zug weiter nach Stade.
- App vs. Wirklichkeit: Der Zug Richtung Stade ist rot, aber leer. Der Gegenzug in Richtung Stadt ist ohne Meldung, aber kontrolliert.
- Folge: Auf dem stadtwärtigen Abschnitt fehlt die Warnung. Pendler am Morgen haben den Schaden.
- Schwere: hoch

### S-09 Langer Zug: Ein Melder sieht nur einen Wagen
- Ablauf: Ein Langzug (Vollzug, 3 Einheiten) auf der S1. A sitzt hinten, F1 steigt vorne ein, F2 hinten. A meldet nur F2. F2 steigt in Landwehr aus, A meldet „ausgestiegen“. F1 ist weiter vorne an Bord.
- App vs. Wirklichkeit: Die App zeigt nach der Entwarnung ⚪. F1 kontrolliert weiterhin.
- Folge: Eine Entwarnung wirkt besonders vertrauenswürdig, ist es aber nicht. B steigt in Hasselbrook ein und wird kontrolliert.
- Schwere: hoch

### S-10 Lücke im Abend- oder Nachtverkehr
- Ablauf: Um 23:40 fahren auf der S3 kaum App-Nutzer. A meldet um 23:41 F1 in Stellingen und steigt in Eidelstedt aus. Danach meldet 40 Minuten lang niemand etwas, weder Bestätigung noch Entwarnung.
- App vs. Wirklichkeit: Die Meldung hängt bis Pinneberg am Zug, Wirklichkeit unbekannt. Andere Züge sind ⚪, weil es keine Nutzer gibt, nicht weil sie frei sind.
- Folge: Bei dünner Datenlage sind sowohl Rot als auch Weiß wertlos. Der Nutzer erkennt den Unterschied nicht.
- Schwere: mittel

### S-11 Zivilkontrolleure sind noch nicht ausgewiesen
- Ablauf: F1 und F2 fahren in Zivil in der S1 und weisen sich erst kurz vor Hasselbrook aus. A war von Altona bis Hbf im Zug und hat nichts gemeldet, weil nichts zu sehen war.
- App vs. Wirklichkeit: Die App zeigt ⚪ und zusätzlich eine „Ich bin in diesem Zug“-Präsenz ohne Meldung. Das wirkt wie eine bestätigte Freiheit.
- Folge: Die App suggeriert Sicherheit durch Präsenz. Fahrgäste ab Berliner Tor haben den Schaden.
- Schwere: hoch

### S-12 Falsche oder missverstandene Meldung
- Ablauf: A hält zwei Sicherheitsleute (DB Sicherheit) in Reeperbahn für Kontrolleure und meldet sie. Niemand widerspricht, weil die anderen es nicht wissen.
- App vs. Wirklichkeit: Die App zeigt „🔴 im Zug“, es gibt aber keine Kontrolle.
- Folge: Viele Nutzer weichen ohne Grund aus und verlieren Zeit. Wiederholt sich das, sinkt die Glaubwürdigkeit.
- Schwere: niedrig

### S-13 Zugtausch oder Ausfall: Meldung wandert ins Leere
- Ablauf: A meldet F1 in der S3 um 14:05 ab Elbbrücken. In Harburg endet der Zug wegen einer Störung vorzeitig, alle steigen in den Folgezug um, F1 auch. Die fahrtKey des alten Zuges ist laut Fahrplan „weiter unterwegs“.
- App vs. Wirklichkeit: Die App zeigt einen Phantomzug in Richtung Neugraben, rot. Der reale Folgezug mit F1 ist ⚪.
- Folge: Bei Störungen, wenn die meisten umsteigen, stimmt die App am wenigsten.
- Schwere: mittel

### S-14 Meldung zur falschen Linie am Gemeinschaftsabschnitt
- Ablauf: A steht in Diebsteich oder an der Sternschanze, wo mehrere Linien halten. A sieht F1 in einen Zug steigen und meldet in Eile die S3, es war aber die S5 (oder eine S7-Fahrt auf demselben Gleis). B wartet auf die S5 und sieht ⚪.
- App vs. Wirklichkeit: Die S3 ist rot, aber leer. Der tatsächliche Zug ist ohne Meldung.
- Folge: Ein Bedienfehler auf Abschnitten mit mehreren Linien erzeugt gleichzeitig falsch-positiv und falsch-negativ.
- Schwere: mittel

## Muster über alle Szenarien

- Meldungen enden nicht von selbst, und Entwarnungen kommen selten (S-01, S-06, S-10).
- „Keine Meldung“ heißt nicht „frei“ (S-02, S-10, S-11).
- Bei Umstiegen, Flügelung, Verspätung und Störungen ordnet die App Meldungen dem falschen Zug zu (S-03, S-04, S-07, S-08, S-13).
- Eine Beobachtung aus einem Wagen wird auf den ganzen Zug übertragen (S-09).
