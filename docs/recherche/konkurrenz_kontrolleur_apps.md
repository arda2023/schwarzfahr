# Konkurrenzanalyse: Warn-Apps und -Kanäle für Fahrkartenkontrollen

Stand: 3. November 2026 · Recherche für Ardas S-Bahn-Kontrolleur-App (Start Hamburg)

> Zahlen stammen aus den verlinkten Quellen (Presse, Telegram-Seiten, App Store, Verzeichnisse wie ChatCrawler). Was ich nicht selbst prüfen konnte, ist mit **(unbestätigt)** markiert. Rechtliches nur, soweit es über Konkurrenten berichtet wurde.

---

## TL;DR

- **Niemand macht es zugbezogen.** Alle Konkurrenten melden pro **Station/Linie/Richtung** oder als Punkt auf einer Karte. Keiner zeigt einzelne laufende Züge zum Durchwischen. Einen Status „Kontrolleure ausgestiegen“ gibt es so auch nirgends.
- **Größter Player in DE: FreiFahren (Berlin).** Hinter der App steht ein Verein. Nach eigener Darstellung ist sie in 13 Städten aktiv, die Telegram-Gruppe hat rund 39.600 Mitglieder, und es gibt eine Risiko-Ampel pro Linie. **Im August 2026 ist die App aus dem App Store verschwunden** und läuft jetzt nur noch als Web-App.
- **Hamburg ist schwach besetzt.** Die Indie-App **KontrollRadar** importiert sogar Facebook-Posts. Die Telegram-Gruppe „Kontrollettis HH“ hat nur 627 Mitglieder, und dort schreibt kaum noch jemand. Eine Facebook-Gruppe hat über 6.000 Mitglieder. Den Markt hat also noch niemand richtig besetzt.
- **Das größte Risiko sind die App-Stores.** FreiFahren (DE 2026) und Akha (Paris 2025) wurden entfernt. PtvAlert (Melbourne) musste sich in den Store-Texten als „Companion-App“ umbenennen.
- **In Frankreich ist das Ganze verboten.** Der Akha-Gründer musste 2025 über 32.000 € Schadenersatz zahlen. Für Deutschland diskutieren Juristen eine **zivilrechtliche Unterlassung** durch Verkehrsbetriebe (beck-aktuell).
- **Die Lehre daraus:** Die App braucht mehr Nutzen als nur Kontrollwarnungen, z. B. Echtzeit-Abfahrten, Störungen und Sicherheit. Dazu kommen Datensparsamkeit, keine Fotos von Kontrolleuren und Web-App als Fallback.

---

## Vergleich auf einen Blick

| App/Kanal | Stadt | Meldung auf | Zug-Tracking |
|---|---|---|---|
| FreiFahren | Berlin + 12 | Station/Linie/Richtung | nein (Risiko-Ampel) |
| KontrollRadar | Hamburg | Linie/Haltestelle | nein |
| Kontrollettis HH | Hamburg | Freitext | nein |
| schwarzkappler.info | Wien | Station, „mobil“/„stationär“ | nein |
| Akha (tot) | Paris/Lyon | Kartenpunkt (GPS) | nein |
| PtvAlert | Melbourne | Kartenpunkt + Linie | nein |
| Memetro | Barcelona/weltweit | Station | nein |
| Range | NL/BE | Kartenpunkt + Richtung | nein |
| FB-Gruppen | HH, Oslo, Stockholm, Melbourne | Freitext | nein |

| App/Kanal | Größe (öffentlich) | Modell | Status |
|---|---|---|---|
| FreiFahren | ~39.600 Telegram, „40.000 regelmäßig“ | Spenden (Verein) | aktiv, iOS entfernt |
| KontrollRadar | keine Zahlen gefunden | Werbung + Supporter | aktiv |
| Kontrollettis HH | 627 Mitglieder | – | kaum aktiv |
| schwarzkappler.info | ~22.000 Telegram | unbekannt | aktiv |
| Akha | 130.000 Nutzer (Eigenangabe) | unbekannt | gelöscht, verurteilt |
| PtvAlert | 10.000 Downloads (04/2025), 4,6★ (146) | Bezahl-App + Banner | aktiv |
| Memetro | keine Zahlen gefunden | – | Web-App aktiv |
| Range | keine Zahlen gefunden | unbekannt | unklar |

---

## Konkurrenten im Detail

### 1. FreiFahren (Berlin, inzwischen mehrere Städte)
- **Links:** [freifahren.org](https://freifahren.org/) · [Web-App](https://app.freifahren.org/) · [Telegram @freifahren_BE](https://t.me/freifahren_BE) · [GitHub](https://github.com/FreiFahren/FreiFahren)
- **Plattformen:** Web-App (PWA), Telegram. Die iOS-App ist weg ([Apple-Link](https://apps.apple.com/de/app/freifahren/id6738277309) liefert 404).
- **Städte:** Laut Website 13. Eigene Telegram-Gruppen gibt es u. a. für München, Leipzig, Köln/VRS, Ruhr/VRR, Hannover, Bremen, Stuttgart, Rostock und Kiel ([ChatCrawler](https://chatcrawler.link/zh/c/t/transport/public?lang=de)). Ob Hamburg dazugehört: **(unbestätigt)**.
- **So funktioniert die Meldung:**
  - Pflichtfeld ist die Station, dazu kommen Linie und Richtung.
  - Alternativ schreibt man frei in die Telegram-Gruppe, und ein Bot extrahiert Station, Linie und Richtung.
  - Die Karte zeigt die Meldungen der letzten Stunde und lädt alle paar Sekunden neu.
  - Ein **Risikomodell** färbt die Linien grün, gelb oder rot und schätzt, wohin sich ein Team vermutlich als Nächstes bewegt ([FAQ](https://freifahren.org/faq/), [Golem](https://www.golem.de/news/freifahren-diese-app-warnt-berliner-fahrgaeste-vor-ticket-kontrollen-2608-211733.html)).
- **Vertrauen und Missbrauch:**
  - Man braucht keinen Account.
  - Eigene Meldungen sind nur alle 15 Minuten möglich.
  - Ein Nähe-Check verlangt, dass man höchstens etwa 1,5 km von der Station entfernt ist.
  - Dazu kommen Cloudflare und ein Spamfilter.
  - Der Standort bleibt auf dem Gerät, an den Server gehen nur Station, Linie, Richtung und Zeit.
- **Zahlen:**
  - Telegram: 39.660 Mitglieder ([t.me](https://t.me/freifahren_BE)).
  - Laut Entwickler rund 40.000 regelmäßige Nutzer und etwa 100 gleichzeitig aktiv ([Golem](https://www.golem.de/news/freifahren-diese-app-warnt-berliner-fahrgaeste-vor-ticket-kontrollen-2608-211733.html)).
  - „Rund 55.000 aktive Melder“ ([beck-aktuell](https://www.beck-aktuell.de/heute-im-recht/rechtsgeschehen/bahn-kontrolle-schwarzfahren-app-berlin-rechtswidrig-2026-08-11)).
  - 2.733 Meldungen in 7 Tagen und 104 Linien laut Website. Eine frühere Angabe nennt 3.518 in 7 Tagen ([iphone-ticker](https://www.iphone-ticker.de/freifahren-berliner-app-warnt-vor-ticketkontrollen-283554/)).
- **Geschäftsmodell:** Ein gemeinnütziger Verein (FreiFahren e.V.) ohne Werbung und ohne Abo, finanziert über Spenden. Das Ganze ist politisch gerahmt, als Protest gegen Ersatzfreiheitsstrafen.
- **Ärger:**
  - Die BVG nennt die App „unsolidarisch“ und prüft rechtliche Schritte.
  - Im August 2026 verschwand die iOS-App. Ein Leserkommentar vom 22.08.2026 meldet die Entfernung. Die FAQ nennt „Probleme mit Apple und Google“ ([iphone-ticker](https://www.iphone-ticker.de/freifahren-berliner-app-warnt-vor-ticketkontrollen-283554/), [FAQ](https://freifahren.org/faq/)).
- **Lehre:** Das Wachstum kam über die bestehenden Chatgruppen, denn die Meldungen wurden zuerst aus Telegram ausgelesen. Die Web-App ist die Rettung bei einer Store-Sperre.

### 2. KontrollRadar (Hamburg)
- **Links:** [App-Info (appstor.io)](https://kontrollradar.appstor.io/) · [Android/Web-Listing (chrome-stats)](https://chrome-stats.com/d/app.fahrkartenkontrolle.meldung)
- **Entwickler:** Alex Opoku (laut Listing).
- **Plattformen:** iOS und Android, Paket `app.fahrkartenkontrolle.meldung`.
- **So funktioniert die Meldung:**
  - Ein Feed mit Meldungen pro **Linie und Haltestelle**.
  - Man kann kommentieren und Meldungen **bestätigen oder entkräften**.
  - Es gibt Favoriten, Push-Mitteilungen (auch für die Nähe) sowie Reputation und Leaderboard.
- **Besonderheiten in Version 1.3.0:**
  - Offizielle **HVV/Geofox-Störungen** erscheinen mit Push.
  - **Facebook-Gruppenposts werden automatisch importiert:** Über Apify werden alle 20 Minuten die zwei neuesten Posts geprüft und von einem Parser in Linie, Haltestelle, Richtung und Anzahl der Kontrolleure übersetzt.
- **Zug-Tracking:** nein.
- **Geschäftsmodell:** Banner-Werbung, gegen „Supporter“-Zahlung werbefrei.
- **Zahlen und Bewertungen:** Habe ich nicht gefunden, die Store-Seite war blockiert **(unbestätigt)**.
- **Lehre:** Das ist der direkte Hamburg-Konkurrent. Er hat bereits Kaltstart-Tricks (FB-Import) und Zusatznutzen (Störungen). Was ihm fehlt: Linienstreifen, Züge und der Status „ausgestiegen“.

### 3. Kontrollettis HH (Telegram, Hamburg)
- **Link:** [ChatCrawler-Profil](https://chatcrawler.link/v/KontrolletisHamburg) (@KontrolletisHamburg)
- **So funktioniert die Meldung:** Als Freitext-Protokoll mit Haltestelle, Linie, Richtung und Anzahl. Die Regeln verbieten Fotos von Kontrolleuren und Beleidigungen.
- **Zahlen:** 627 Mitglieder. In den letzten 30 Tagen gab es 8 Nachrichten von 1 schreibenden Person.
- **Lehre:** In Hamburg fehlt eine aktive Community. Die Chance besteht, sie aufzubauen, aber dafür muss man den Kaltstart lösen.

### 4. schwarzkappler.info (Wien)
- **Links:** [schwarzkappler.info](https://schwarzkappler.info/) · [Telegram @schwarzkapplerinfo](https://t.me/schwarzkapplerinfo) · [Statistik](https://www.schwarzkappler.info/statistik)
- **So funktioniert die Meldung:**
  - Pro Linie und Station mit Typ **„Mobil“ (fährt mit, z. B. „Siebensterngasse → Skodagasse“) oder „Stationär“** und Ortsdetail (z. B. „Übergang U1 nach U2“).
  - Nutzer können „**Noch aktiv**“ oder „**Vorbei**“ klicken, außerdem kommentieren.
  - Dazu kommen Störungen der Wiener Linien und eine Statistik der meistkontrollierten Linien.
- **Zahlen:** Telegram hat 22.039 Mitglieder (t.me, 11/2026).
- **Lehre:** Das kommt Ardas Idee am nächsten. „Mobil“ mit Richtung und der Status „Vorbei“ gibt es hier schon, aber nicht auf einzelne Züge bezogen.

### 5. Akha (Paris/Île-de-France, Lyon) – eingestellt
- **Quellen:** [Le Figaro (Entfernung)](https://www.lefigaro.fr/secteur/high-tech/valerie-pecresse-veut-faire-fermer-akha-une-application-qui-signale-la-presence-de-controleurs-ratp-20250113) · [Capital](https://www.capital.fr/economie-politique/akha-critique-pour-son-signalement-des-controleurs-ratp-et-sncf-a-son-tour-banni-de-l-app-store-1508261) · [Le Figaro (Urteil)](https://www.lefigaro.fr/secteur/high-tech/le-createur-de-l-application-akha-qui-signalait-la-presence-de-controleurs-ratp-et-sncf-condamne-au-civil-20250513) · [actu.fr](https://actu.fr/ile-de-france/paris_75056/son-application-permettait-d-echapper-aux-controleurs-sncf-et-ratp-le-createur-d-akha-lourdement-condamne_62635280.html)
- **So funktioniert die Meldung:** Wie Waze: eine Karte mit Geolokalisierung und drei Meldetypen, nämlich Verspätung, Unsicherheit und „akha“ (= Kontrolleure).
- **Zahlen:** Gestartet Anfang 2024, laut Eigenangabe 130.000 Nutzer. Zeitweise war sie die meistgeladene App in der Kategorie „Transport“.
- **Rechtlicher Ärger:**
  - Im Januar 2025 entfernten Google Play und danach Apple die App, auf Druck von Valérie Pécresse und IDFM.
  - IDFM stellte Strafanzeige, der Gründer kam in Polizeigewahrsam, und die Website wurde beschlagnahmt.
  - Am 7.5.2025 wurde er verurteilt, insgesamt 32.020 € Schadenersatz an IDFM, RATP und SNCF zu zahlen.
  - Grundlage ist Art. L2242-10 Code des transports. In Frankreich ist das Melden von Kontrolleuren grundsätzlich verboten.
- **Lehre:** Viralität ohne Rechtsrahmen bedeutet den Tod. Der Name und das „Augenzwinkern“ (Drogen-Slang) haben Politik und Presse zusätzlich provoziert.

### 6. CheckMyMetro (Paris, 2011–ca. 2014) – historisch
- **Quellen:** [Journal du Net](https://www.journaldunet.com/ebusiness/internet-mobile/1089869-la-ratp-demande-a-apple-de-supprimer-l-appli-communautaire-checkmymetro/) · [Numerama](https://www.numerama.com/politique/19126-la-ratp-demande-a-apple-de-proteger-son-monopole-sur-iphone.html) · [20 Minutes](https://www.20minutes.fr/paris/1455527-20141007-check-my-metro-application-dit-o-controleurs)
- **Was es war:** Eine Community-App mit „Checks“ zu Musikern, Graffiti und Kontrolleuren, dazu Punkte und Gamification. Geplant war ein Preis von 1,79 €.
- **Ärger:** 2011 verlangte die RATP von Apple die Löschung, und zwar **nicht wegen der Kontrolleure**, sondern weil **Metro-Karte und Fahrplandaten urheberrechtlich geschützt** seien.
- **Lehre:** Für Ardas Linienstreifen mit Abfahrtszeiten müssen die Datenlizenzen sauber sein (offene GTFS-Daten bzw. HVV-API-Bedingungen). Keine offiziellen Netzpläne oder Logos verwenden.

### 7. PtvAlert (Melbourne)
- **Links:** [App Store](https://apps.apple.com/au/app/ptvalert/id6736619184) · [Information Age](https://ia.acs.org.au/article/2025/popular-app-tracks-myki-ticket-inspectors.html)
- **So funktioniert die Meldung:**
  - Ein Kartenpunkt mit Kategorie und Freitext. Ab Version 1.6 lassen sich auch Verkehrsmittel und Tram- oder Zuglinie angeben.
  - Ältere Meldungen verblassen, nach **20–30 Minuten** verschwinden sie.
  - Anfangs war 1 Meldung pro 12 Stunden erlaubt.
  - Es gibt Sign-in mit Apple und eine Anzeige von „Routen mit vielen Alerts“.
- **Zahlen:**
  - 10.000 Downloads in wenigen Tagen nach einem Reddit-Post mit rund 1.100 Upvotes.
  - Platz 6 in der Kategorie Navigation (AU).
  - 4,6★ bei 146 Bewertungen.
  - Laut Entwickler mehr als 600 Meldungen pro Tag.
  - Über 98 % der Meldungen betreffen Kontrolleure, obwohl es offiziell drei Kategorien gibt.
- **Geschäftsmodell:** Banner-Werbung, später Bezahlschranke (2 AUD, In-App „Full Access“ 4,99 AUD). Nutzer beschweren sich, dass dadurch weniger gemeldet wird.
- **Store-Ärger:** In den Release Notes hat der Entwickler die App gegenüber Apple umgedeutet zur „Public Transport Companion App“, die „keine Schwarzfahrt fördert“. Kontrolleure werden euphemistisch „wildlife“ genannt.
- **Lehre:**
  - Reddit oder Uni-Communities zünden schnell.
  - Eine Paywall vor nutzergenerierten Daten schadet dem Netzwerkeffekt.
  - Store-Reviewer schauen genau hin.

### 8. Facebook-Gruppen in Melbourne („myki inspector alert space“)
- **Quellen:** [Information Age](https://ia.acs.org.au/article/2025/popular-app-tracks-myki-ticket-inspectors.html) · [Guardian 2015](https://www.theguardian.com/australia-news/2015/dec/02/where-are-our-mates-facebook-group-keeps-tabs-on-melbournes-tram-inspectors) · [BBC](https://www.bbc.com/news/blogs-trending-37197751)
- **Größe:** über 133.400 Mitglieder (2025).
- **Lehre:** Facebook und Telegram sind die eigentliche „Konkurrenz“, denn dort sitzt die Masse der Nutzer. Eine App muss deutlich bequemer sein als ein Post.

### 9. Memetro / Cola't (Barcelona, „jede Stadt“)
- **Links:** [memetro.org](https://memetro.org) · [Telegram @AvisosControlsBCN](https://t.me/AvisosControlsBCN) · [MemetroMap (eingestellt)](https://www.mapametro.com/memetrobcnmap/)
- **So funktioniert die Meldung:** Über eine Web-App (ohne Installation), eine offene Telegram-Gruppe und einen **anonymen Melde-Bot** (@ColatBarcelonaBot).
- **Lehre von MemetroMap:** Diese Karte wurde aus Tweets gespeist. „**Abgesagte Alarme**“ („Station frei“) wurden nur markiert und nicht gelöscht, weil sonst Kontrolleure selbst Entwarnung geben könnten. Das ist wichtig für Ardas „ausgestiegen“-Status.
- **Zahlen:** keine gefunden. Madrid hat laut [El País](https://elpais.com/espana/madrid/2022-08-19/asi-se-organizan-en-las-redes-sociales-los-antisistemas-del-metro-de-madrid-para-colarse-sin-pagar-el-billete.html) ähnliche Telegram-Gruppen.

### 10. Range (Niederlande/Belgien)
- **Quelle:** [Androidworld 2020](https://androidworld.nl/apps/range-app-controles-verkeer/)
- **Was es ist:** Eine Allround-„Kontrollen“-App für Polizei, Blitzer und auch **ÖV-Kontrollen**. Entstanden ist sie aus WhatsApp-Gruppen in Rotterdam.
- **So funktioniert die Meldung:**
  - Man meldet per GPS, und das nur in der eigenen Nähe.
  - Dazu kommen Richtung und Beschreibung.
  - Die App zeigt „wie viele Nutzer bestätigt“ und „letzte Meldung“ an.
- **Status 2026:** **(unbestätigt)**.

### 11. Skandinavien
- **Stockholm:** [Planka.nu](https://planka.nu/plankning/stockholm/) betreibt eine „Schwarzfahrkasse“ und verweist auf die FB-Gruppe „Här befinner SL-kontrollanterna sig“ sowie den Hashtag #biljettkontroll.
  - **2018 hat SL eine Frau über ihren Facebook-Post identifiziert und ihr eine Strafgebühr geschickt**, obwohl sie nach eigener Aussage bezahlt hatte und nur andere warnte ([Planka.nu](https://planka.nu/2018/02/15/biljettkontroll-pa-facebook-avgiftssamhallet-sparar-ur/)).
  - Lehre: Klarnamen-Plattformen gefährden die Melder, deshalb gehört Anonymität ins Kernfeature.
- **Oslo:** Es gibt Facebook-Gruppen, die vor Kontrollen warnen ([Avisa Oslo](https://www.ao.no/advarer-mot-det-som-er-i-ferd-med-a-skje-pa-bussene-viser-fingeren-og-sier-du-kan-ikke-gjore-meg-en-dritt/s/5-128-982589)). Die Mitgliederzahl, laut Suchergebnis „über 33.000“, ist **(unbestätigt)**.

### 12. Weitere deutsche Telegram-Gruppen (Auswahl, ChatCrawler 11/2026)
| Gruppe | Region | Mitglieder |
|---|---|---|
| @freifahren_leipzig | Leipzig | 7.526 |
| @BVG_SBAHN_3G_Kontrolle | Berlin | 3.371 |
| @freifahren_st | Stuttgart | 2.739 |
| @freifahren_mu | München | 2.715 |
| @Freifahren_VRS | Köln/Bonn | 2.107 |
| @ControlletispyHB | Bremen | 1.873 |
| @freifahren_h | Hannover | 1.071 |
| @freifahren_VRR | Ruhr | 865 |
| @KontrolletisHamburg | Hamburg | 627 |

Quelle: [ChatCrawler Kategorie ÖPNV](https://chatcrawler.link/zh/c/t/transport/public?lang=de). In München gab es früher außerdem „MVV Blitzer“ auf Facebook und Twitter ([Abendzeitung](https://www.abendzeitung-muenchen.de/muenchen/mvv-kontrolle-so-wird-im-internet-gewarnt-art-166586), [Merkur](https://www.merkur.de/lokales/muenchen/informatik-student-warnt-schwarzfahrer-kontrolleuren-1220818.html)).

---

## Hamburg im Fokus

- **KontrollRadar** ist der einzige echte App-Konkurrent. Er ist technisch schon recht weit: HVV/Geofox-Störungen, FB-Import, Reputation und Bestätigen/Entkräften. Was fehlt: eine Linien- und Zugansicht.
- **Eine Facebook-Gruppe mit über 6.000 Mitgliedern** warnt vor HVV-Kontrollen. Am häufigsten werden dort zuletzt **S1 und U2** genannt ([moin.de, 02.03.2026](https://www.moin.de/norddeutschland/hvv-schwarzfahrer-gruppen-warnungen-facebook-organisieren-id300837690.html)). Den Gruppennamen nennt der Artikel nicht.
- **Telegram „Kontrollettis HH“:** 627 Mitglieder, praktisch inaktiv.
- **Die hvv-App** warnt nicht vor Kontrollen, sondern nur vor Störungen. Die S-Bahn Hamburg hat einen eigenen [Störungsalarm](https://www.s-bahn-hamburg.de/stoerungsalarm).
- **Der HVV kündigt manchmal selbst Kontrollen an**, etwa beim „Prüfmarathon“. Trotz Ankündigung erwischte er dabei Hunderte ohne gültiges Ticket ([Abendblatt](https://www.abendblatt.de/hamburg/politik/article413233135/hvv-trotz-warnung-hunderte-schwarzfahrer-bei-pruefmarathon-oft-liegt-es-am-handy.html), [r/hamburg](https://www.reddit.com/r/hamburg/comments/1nnx4d7/hvv_fahrkartenkontrollen_pr%C3%BCfmarathon_am_25092025/)).
- **Stimmung auf Reddit:**
  - Diskutiert wird über aggressive Kontrolleure ([Thread](https://www.reddit.com/r/hamburg/comments/1bsxbyx/die_fahrkartenkontrolleure_der_hvv_wirken/)).
  - Ein anderer Thread fragt, ob Kontrolleure echt sind ([Thread](https://www.reddit.com/r/hamburg/comments/1wjylp3/hvvkontrolleure_wirklich_echt/)).
  - Das zeigt ein Bedürfnis nach Echtheits- und Sicherheitsinfos, nicht nur nach Warnungen.
- **Initiative „Kontrolleur:innen kontrollieren“** (Wilhelmsburg): Sie protestiert gegen häufige Abgangskontrollen an den S-Bahn-Stationen **Veddel und Wilhelmsburg** (S3/S5), 2023 gab es zwei Demos ([InselRundblick](https://inselrundblick.de/aktuell/weil-sie-es-koennen/)). Das ist ein Hinweis auf Hotspots und auf stationäre Kontrollen, die nicht im Zug stattfinden.
- **Rechtliches (nur Presse zu Konkurrenten):** Ein Anwalt (Solmecke) hält das allgemeine Warnen für legal, **Fotos von Kontrolleuren und HVV-Logos** dagegen für problematisch (moin.de).

---

## Lücken, die Ardas Konzept füllt

1. **Zugbezogene Meldungen.** Alle melden auf Station oder Linie. Nur FreiFahren *schätzt* die Bewegung, und Wien kennt „mobil A → B“. Eine Meldung, die an einer konkreten Fahrt hängt („S1 um 08:12 ab Ohlsdorf Richtung Wedel“) und mit dem Zug „mitwandert“, gibt es nirgends.
2. **Linie als vertikaler Stationsstreifen mit Abfahrtszeiten.** Die Konkurrenten zeigen Karten (Akha, PtvAlert, FreiFahren) oder Feeds (KontrollRadar). Eine Linienansicht passt besser zur S-Bahn-Realität.
3. **Zwischen laufenden Zügen wischen.** Gibt es nirgends. Damit lassen sich „der Zug vor mir“ und „der Zug hinter mir“ vergleichen.
4. **Status „Kontrolleure ausgestiegen“ / „ich habe den Zug verlassen“.** Wien hat nur „Vorbei“, KontrollRadar nur „Entkräften“. Ein Ausstiegsort, der den Zug wieder „grün“ macht und die Station rot färbt, ist neu.
5. **Optionales Tracking des Melders.** Wenn der Melder mitfährt, bestätigt das die Meldung passiv. Zum Vergleich: FreiFahren prüft nur 1,5 km Nähe, PtvAlert nur den Ort.
6. **Hamburg-Community** ist noch schwach, nur KontrollRadar ist aktiv.

---

## Lehren und Risiken aus der Konkurrenz

- **Store-Sperre einplanen:**
  - FreiFahren (iOS 2026), Akha (beide Stores 2025) und CheckMyMetro (Löschantrag 2011) zeigen das Risiko.
  - Deshalb **von Anfang an PWA/Web-App** bauen.
  - Store-Texte auf Echtzeit, Störungen und Sicherheit ausrichten (vgl. PtvAlert). Wichtig: Wenn der Store-Text etwas anderes verspricht, als die App tut, kann das selbst ein Verstoß gegen die Store-Regeln sein. Das hier ist nur eine Beobachtung, keine Rechtsberatung.
- **Zusatznutzen** wie Störungen oder Abfahrten machen die App alltagstauglich und weniger angreifbar (KontrollRadar, Wien).
- **Datenrechte:** Netzpläne, Logos und Fahrplandaten nur mit passender Lizenz nutzen (CheckMyMetro).
- **Anonymität und Datensparsamkeit:**
  - Die Stockholmer SL hat einen Melder über Facebook identifiziert.
  - Die Pariser Ermittler wollten die beschlagnahmten Daten zur „Identifizierung der Kunden“ auswerten.
  - Vorbild FreiFahren: Der Standort bleibt auf dem Gerät.
- **Missbrauch:** In den Kommentaren unter Artikeln wird offen dazu aufgerufen, die App mit Fake-Meldungen zu fluten ([iphone-ticker-Kommentare](https://www.iphone-ticker.de/freifahren-berliner-app-warnt-vor-ticketkontrollen-283554/)). Barcelona hat Entwarnungen bewusst nicht als Löschung behandelt. Gegenmittel:
  - Rate-Limit
  - Nähe-Check
  - Reputation
  - Mehrfachbestätigung
- **Ablauf von Meldungen:** Üblich sind 20–60 Minuten (PtvAlert 20–30, FreiFahren 60). Ältere Meldungen sollten verblassen.
- **Kaltstart:** Fast alle Apps sind aus Chatgruppen gewachsen. FreiFahren und KontrollRadar importieren Posts von dort. Auch ein Reddit-Post kann einen Start auslösen (PtvAlert).
- **Monetarisierung:** Vereine leben von Spenden, Indie-Apps von Werbung oder Supporter-Modell. Eine harte Paywall bremst die Meldungen (Kritik an PtvAlert).
- **Ton und Branding:** Provokante Namen (Akha) haben Politik und Medien befeuert. Ein neutraler Name und Regeln wie „keine Fotos, keine Beleidigungen“ schützen.

---

## Quellen

- FreiFahren: [Website](https://freifahren.org/) · [FAQ](https://freifahren.org/faq/) · [Expansion](https://freifahren.org/expansion/) · [GitHub](https://github.com/FreiFahren/FreiFahren) · [Telegram](https://t.me/freifahren_BE)
- [Golem, 08.08.2026](https://www.golem.de/news/freifahren-diese-app-warnt-berliner-fahrgaeste-vor-ticket-kontrollen-2608-211733.html)
- [iphone-ticker, 17.08.2026](https://www.iphone-ticker.de/freifahren-berliner-app-warnt-vor-ticketkontrollen-283554/)
- [beck-aktuell, 11.08.2026](https://www.beck-aktuell.de/heute-im-recht/rechtsgeschehen/bahn-kontrolle-schwarzfahren-app-berlin-rechtswidrig-2026-08-11)
- KontrollRadar: [appstor.io](https://kontrollradar.appstor.io/) · [chrome-stats](https://chrome-stats.com/d/app.fahrkartenkontrolle.meldung)
- [Kontrollettis HH (ChatCrawler)](https://chatcrawler.link/v/KontrolletisHamburg) · [ChatCrawler ÖPNV-Ranking](https://chatcrawler.link/zh/c/t/transport/public?lang=de)
- [moin.de, 02.03.2026](https://www.moin.de/norddeutschland/hvv-schwarzfahrer-gruppen-warnungen-facebook-organisieren-id300837690.html)
- [InselRundblick Wilhelmsburg](https://inselrundblick.de/aktuell/weil-sie-es-koennen/)
- [Abendblatt Prüfmarathon](https://www.abendblatt.de/hamburg/politik/article413233135/hvv-trotz-warnung-hunderte-schwarzfahrer-bei-pruefmarathon-oft-liegt-es-am-handy.html)
- schwarzkappler.info: [Website](https://schwarzkappler.info/) · [Telegram](https://t.me/schwarzkapplerinfo)
- Akha: [Le Figaro (Entfernung)](https://www.lefigaro.fr/secteur/high-tech/valerie-pecresse-veut-faire-fermer-akha-une-application-qui-signale-la-presence-de-controleurs-ratp-20250113) · [Le Figaro (Urteil)](https://www.lefigaro.fr/secteur/high-tech/le-createur-de-l-application-akha-qui-signalait-la-presence-de-controleurs-ratp-et-sncf-condamne-au-civil-20250513) · [Capital](https://www.capital.fr/economie-politique/akha-critique-pour-son-signalement-des-controleurs-ratp-et-sncf-a-son-tour-banni-de-l-app-store-1508261) · [actu.fr](https://actu.fr/ile-de-france/paris_75056/son-application-permettait-d-echapper-aux-controleurs-sncf-et-ratp-le-createur-d-akha-lourdement-condamne_62635280.html)
- CheckMyMetro: [JDN](https://www.journaldunet.com/ebusiness/internet-mobile/1089869-la-ratp-demande-a-apple-de-supprimer-l-appli-communautaire-checkmymetro/) · [Numerama](https://www.numerama.com/politique/19126-la-ratp-demande-a-apple-de-proteger-son-monopole-sur-iphone.html) · [BeGeek](https://www.begeek.fr/lapplication-checkmymetro-connecte-les-voyageurs-du-metro-9068)
- PtvAlert: [App Store](https://apps.apple.com/au/app/ptvalert/id6736619184) · [Information Age](https://ia.acs.org.au/article/2025/popular-app-tracks-myki-ticket-inspectors.html)
- Melbourne: [Guardian 2015](https://www.theguardian.com/australia-news/2015/dec/02/where-are-our-mates-facebook-group-keeps-tabs-on-melbournes-tram-inspectors)
- Memetro: [memetro.org](https://memetro.org) · [Telegram BCN](https://t.me/AvisosControlsBCN) · [MemetroMap](https://www.mapametro.com/memetrobcnmap/)
- Madrid: [El País](https://elpais.com/espana/madrid/2022-08-19/asi-se-organizan-en-las-redes-sociales-los-antisistemas-del-metro-de-madrid-para-colarse-sin-pagar-el-billete.html)
- Range: [Androidworld](https://androidworld.nl/apps/range-app-controles-verkeer/)
- Skandinavien: [Planka.nu 2018](https://planka.nu/2018/02/15/biljettkontroll-pa-facebook-avgiftssamhallet-sparar-ur/) · [Avisa Oslo](https://www.ao.no/advarer-mot-det-som-er-i-ferd-med-a-skje-pa-bussene-viser-fingeren-og-sier-du-kan-ikke-gjore-meg-en-dritt/s/5-128-982589)

*Nicht gefunden bzw. nicht prüfbar:* eine App namens „Check-Point“, Bewertungen von KontrollRadar (Store-Seite blockiert), der Name der Hamburger Facebook-Gruppe, aktuelle Range-Zahlen. Bei Reddit und Instagram habe ich nur Suchergebnisse gesehen, keine vollständigen Threads.
