# Problem-Szenarien: Übersicht

Die Ergebnisse sind in [KONZEPT.md Version 2](../KONZEPT.md) eingearbeitet. Die Szenarien sind fiktiv.

Fünf Agenten haben Szenarien erzeugt (schnell, niedriger Aufwand). Danach haben fünf weitere Agenten mit hohem Aufwand für jedes Szenario eine Lösung gesucht. Faktenfehler in den Szenarien sind in den Lösungsdateien als *Korrektur* vermerkt.

| Gruppe | Szenarien | Lösungen | ✅ | 🟡 | ❌ |
|---|---|---|---|---|---|
| S – Veraltete und falsche Meldungen | [01](01_veraltete_meldungen.md) | [Lösungen](loesungen_01_veraltete_meldungen.md) | 3 | 9 | 2 |
| Z – Zugzuordnung | [02](02_zugzuordnung.md) | [Lösungen](loesungen_02_zugzuordnung.md) | 5 | 10 | 0 |
| M – Missbrauch und Angriffe | [03](03_missbrauch.md) | [Lösungen](loesungen_03_missbrauch.md) | 3 | 9 | 1 |
| K – Verhalten der Kontrolleure | [04](04_kontrolleur_verhalten.md) | [Lösungen](loesungen_04_kontrolleur_verhalten.md) | 4 | 8 | 0 |
| T – Technik, Nutzerdichte, Bedienung | [05](05_technik_ux.md) | [Lösungen](loesungen_05_technik_ux.md) | 6 | 6 | 1 |
| **Summe** | **67** | | **21** | **42** | **4** |

## Leitfrage S-01: „Eigenes Risiko?“

Ja. A meldet in Rissen, steigt in Blankenese aus, F1 steigt unbemerkt aus. Die App kann nicht wissen, was niemand sieht. Sie kann aber verhindern, dass eine veraltete Meldung wie eine sichere aussieht:

- Steigt der einzige Melder aus, ist der Zug nicht mehr 🔴, sondern 🟠: „zuletzt gesehen Rissen, Melder seit Blankenese nicht mehr an Bord“.
- Die Meldung wird **an jedem Halt** schwächer (nicht nach Minuten), weil Kontrolleure an Halten aussteigen. Nach einigen Halten ohne Bestätigung ist sie ⚪.
- Eingecheckte Nutzer im selben Zug bekommen eine Ein-Tap-Rückfrage („Kontrolleure noch da?“).
- B sieht neben der Warnung den nächsten Zug. Die Entscheidung bleibt bei B.

## Nicht lösbar (❌), nur ehrlich darstellbar

1. **S-02 stilles Zusteigen** und **S-11 / K-01 Zivilkontrolleure:** Was keiner sieht oder erkennt, kann keiner melden.
2. **T-09 Großeinsatz ohne Meldungen:** Wenn alle gleichzeitig kontrolliert werden, meldet niemand.
3. **M-05 Mitlesen durch das Verkehrsunternehmen:** Was die App anzeigt, kann jeder archivieren. Man kann nur wenig preisgeben (keine Rohdaten, keine IDs, keine Historie).

Weitere Teilfälle ohne Lösung: Kurzwende ohne Melder und ohne Echtzeit (Z-03), unbeobachteter Wechsel des Zugteils in Ohlsdorf (Z-04, K-02), einzelne Ersatzbusse (Z-13), Gegner mit mehreren alten Geräten im echten Zug (K-09).

## Die wichtigsten Änderungen am Konzept (aus allen fünf Gruppen)

1. **⚪ heißt „keine Meldung“, nie „frei“.** Es kommt eine zweite Achse dazu, die Datenlage: „keine Daten“, „wenige / einige / viele Nutzer an Bord“. Wörter wie „frei“ und „sicher“ sind verboten. Status immer als Form + Text + Farbe (S, K, T).
2. **Wahrscheinlichkeit statt Ampel-Timer.** Pro Zug und Zugteil, abklingend pro Halt, 🔴 nur bei frischer Sichtung. An Umstiegsknoten und Endstationen gibt es einen Hinweis „möglich“ für Anschluss- und Gegenzüge (S, K).
3. **Warnung und Entwarnung ungleich gewichten.** `AUSGESTIEGEN_IN` macht aus 🔴 höchstens 🟠. Neutral wird ein Zug nur nach Fahrtende, Zeitablauf oder mehreren unabhängigen Entwarnungen (S, M, K).
4. **Eine Meldung ist eine Beobachtung mit Anker** (Station, Zeitpunkt der Beobachtung, gesehenes Ziel, Zugteil). Die Zuordnung zur Fahrt lässt sich später korrigieren. Check-in über das Ziel an der Zuganzeige statt „nächster Zug laut Fahrplan“ (Z, T).
5. **Zug, Fahrt und Kopplung trennen** (Flügelung, Vereinigung, S5-Durchbindung, S7 in Altona, Wende). Neuer `fahrtKey` mit Ziel, versionierte Fahrplandatensätze mit Ablaufdatum (Z).
6. **Server berechnet den Status, Rohmeldungen sind nicht lesbar.** Geprüfte Geräte (App Attest / Play Integrity), täglich wechselnde Pseudonyme, Löschung kurz nach Ereignisende statt nach 24 h, Serverzeit statt Handy-Uhr (M, T).
7. **Sicherheitsregeln:** kein Push für den eigenen oder gleich einfahrenden Zug, keine Ausweichempfehlungen (M-10, Gedränge an Türen).
8. **Echtzeit früher einplanen:** Für einen Test auf der S1 reicht der Fahrplan. Ab dem öffentlichen Launch sollte die DB Timetables API mit Cache-Server da sein (Phase 3 → Phase 2). Messbarer Auslöser: mehr als 10 % der Check-ins in der HVZ brauchen eine Korrektur (Z).

Offen für vor dem Launch: Zuordnung vorne/hinten in Ohlsdorf vor Ort prüfen; aktuelle Supabase-Limits und Preise prüfen.
