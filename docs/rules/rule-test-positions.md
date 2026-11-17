# Positionsbasierter Regeltestkatalog

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Notation und Testarten

R/B bezeichnen Rot/Schwarz, G General,A Advisor,E Elephant,H Horse,R Chariot,C Cannon,S Soldier. Koordinaten sind(x,y). Für reine Movement-Tests gelten nur aufgelistete Figuren und pseudo_legal; fehlende Generäle sind hier absichtlich außerhalb des untersuchten Bausteins. Für Legal-/Outcome-Tests Basis F verwenden: RG(4,0),BG(4,9),RS(4,5), Rot am Zug, sofern nicht überschrieben. Zusätze dürfen keinen Basisplatz doppelt belegen. Full-State-Validator prüft genau einen General pro Seite, Koordinaten, Materialobergrenzen und General-Sicherheit des gerade erfolgten Zuges getrennt von der Movement-API.

| ID | Art und Stellung | Aktion/Frage | Erwartung |
|---|---|---|---|
| P01 | Movement RG(4,0) | nach(4,1) | pseudo-legal |
| P02 | RG(3,0) | nach(2,0) | Palastgrenze abweisen |
| P03 | RG(4,1) | nach(5,2) | diagonalen Generalzug abweisen |
| P04 | RA(4,1) | nach(5,2) | erlaubt |
| P05 | RA(3,0) | nach(2,1) | außerhalb Palast abweisen |
| P06 | RE(2,0) | nach(4,2) | erlaubt |
| P07 | RE(2,0),RS(3,1) | nach(4,2) | Elephant Eye blockiert |
| P08 | RE(2,4) | nach(4,6) | Fluss abweisen |
| P09 | BE(2,5) | nach(4,3) | gespiegelte Flussgrenze abweisen |
| P10 | RH(4,4) | nach(6,5) | erlaubt |
| P11 | RH(4,4),RS(5,4) | nach(6,5) | Horse Leg blockiert |
| P12 | RH(4,4),RS(4,5) | nach(6,5) | erlaubt; falsches Bein wäre Bug |
| P13 | RH(4,4),RS(4,5) | nach(5,6) | blockiert |
| P14 | RR(0,0),RS(0,2) | nach(0,3) | eigener Blocker stoppt |
| P15 | RR(0,0),BS(0,2) | nach(0,2) | schlagen erlaubt |
| P16 | RR(0,0),BS(0,2) | nach(0,3) | gegnerischen Blocker nicht überspringen |
| P17 | RC(1,2) | nach(1,6), leer | ruhiger Zug erlaubt |
| P18 | RC(1,2),BS(1,6) | nach(1,6) | ohne Screen kein Schlag |
| P19 | RC(1,2),RS(1,4),BS(1,6) | nach(1,6) | genau ein Screen erlaubt |
| P20 | RC(1,2),BS(1,4),BS(1,6) | nach(1,6) | gegnerischer Screen ebenfalls erlaubt |
| P21 | RC(1,2),RS(1,3),BS(1,4),BS(1,6) | nach(1,6) | zwei Screens verbieten Schlag |
| P22 | RC(1,2),RS(1,4) | nach(1,6), leer | Screen verbietet ruhigen Zug |
| P23 | RS(0,3) | nach(1,3) | vor Fluss seitwärts verboten |
| P24 | RS(0,5) | nach(1,5) | nach Fluss seitwärts erlaubt |
| P25 | RS(0,5) | nach(0,4) | rückwärts verboten |
| P26 | BS(8,4) | nach(7,4) | schwarz nach Fluss seitwärts erlaubt |
| P27 | RS(0,9) | nach(1,9) | seitwärts ohne Promotion |
| P28 | F, ersetze RS(4,5) durch RR(4,5) | RR nach(3,5) | Flying General öffnet: illegal |
| P29 | F plus BR(3,0),RS(0,3) | RS nach(0,4) | bestehendes Schach bleibt: illegal |
| P30 | F plus BR(3,0) | RG nach(4,1) | Flucht aus Chariot-Linie erlaubt |
| P31 | F, Schwarz am Zug, RR(3,8),RR(5,8) | Outcome | stalemate, Rot gewinnt |
| P32 | P31 plus RS(4,8) | Outcome | checkmate, Rot gewinnt |
| P33 | P31 ohne RR(5,8) | BG nach(5,9) | Flucht legal, kein Endergebnis |
| P34 | F, BG zusätzlich(3,9) statt ersetzt | State laden | doppelter General: ungültig |
| P35 | F plus RC(4,3),BH(4,7) | RS(4,5) seitwärts nach(5,5) | Cannon bleibt Blocker; General nicht allein deshalb offen |
| P36 | F plus eigener RS(0,3) | nach(0,3) als Nullzug | abweisen, Original unverändert |

P35 dient nur der General-Linie; prüfe sämtliche anderen Angriffe, bevor ein globales Legalitätsurteil behauptet wird. Die Fixture-API muss Start/Ziel-Kollisionen verhindern. Jeder Fall wird für Schwarz gespiegelt und mit Randkoordinaten kombiniert.

## Zusätzliche Sequenzen und Eigenschaften

Repetitionfälle aus [Repetition](xiangqi-repetitions.md) mit kompletter Herkunftshistory ausführen. Ein Move darf Material nur um0 oder1 reduzieren, niemals eigene Figur schlagen und genau ein Zugrecht wechseln. apply(rejected) lässt Position bitweise beziehungsweise semantisch gleich. Alle von legal_moves gelieferten Züge werden von validate_move akzeptiert. Nach Probezug bleibt ursprüngliches Brett unverändert. Gleichwertige Positionen liefern unabhängig von Figurenspeicherreihenfolge dieselben Züge.

Für Flying General mindestens drei Fälle: letzter Blocker entfernt, neuer Blocker eingesetzt, General wechselt Datei. Für Cannon Check: null/ein/zwei Screens und Screen wird durch die untersuchte Aktion selbst verändert. Für Horse Check: Bein wird geöffnet und geschlossen. Für Matt/Patt nicht nur Ergebnis testen, sondern vollständige Liste legaler Antworten leer verlangen.

## Qualitätsgrenze

Diese36 Basisfixtures plus Spiegelung sind Mindestfälle, kein Vollständigkeitsbeweis der Regeln. Property-Based-Tests erzeugen gültige Zustände aus legalen Zugfolgen; zufällige Brettbelegung produziert überwiegend ungültige Stellungen. Fehlerseed aufbewahren und minimieren. Bevor ein Turnierprofil eingeführt wird, braucht jede zusätzliche WXF-Schiedskategorie eigene Quellenstelle, Position, Gegenbeispiel und Review.
