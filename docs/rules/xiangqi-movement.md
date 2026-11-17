# Alle sieben Figuren

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Pseudo-legal versus legal

Pseudo-legal prüft Figurengeometrie, Blocker und erlaubte Zielbelegung. Legal prüft zusätzlich nach einem isolierten Probezug, ob der eigene General bedroht ist. Die Engine arbeitet mit unveränderlichem Eingabebrett oder sicher rückgängig gemachtem lokalen Probeobjekt; ein abgelehnter Zug verändert nie die Originalposition.

| Figur | Geometrie | Einschränkung | Schlagverhalten |
|---|---|---|---|
| General | ein orthogonaler Schritt | eigener Palast | wie Bewegung; Flying General als zusätzliche Angriffslinie |
| Advisor | ein diagonaler Schritt | eigener Palast | wie Bewegung |
| Elephant | zwei diagonal | roter Zielpunkt y<=4, schwarzer y>=5; Mittelpunkt leer | wie Bewegung |
| Horse | (2,1) oder(1,2) | orthogonaler erster Schritt in Richtung der langen Achse frei | springt nicht über Horse-Leg |
| Chariot | beliebig weit orthogonal | alle Zwischenpunkte leer | erste gegnerische Figur auf Linie schlagbar |
| Cannon | beliebig weit orthogonal | Nichtschlag:0 Screens; Schlag:genau1 Screen zwischen Start/Ziel | Screen beliebiger Farbe; Ziel gegnerisch |
| Soldier | ein vorwärts | Rot+y, Schwarz−y | nach Flussquerung zusätzlich seitwärts, nie rückwärts |

River-crossed wird aus der aktuellen Position abgeleitet: Rot y>=5, Schwarz y<=4. Es gibt keine Umwandlung. Auf letzter Reihe kann Soldier nur seitwärts, sofern Fluss überquert. Cannon darf nicht auf eigenem Ziel landen und nicht den Screen selbst „überspringend entfernen“. Horse-Leg wird nicht aus dem Ziel rückwärts geraten: von(4,4) nach(6,5) ist(5,4) das Bein, nach(5,6) ist(4,5) das Bein. Elephant von(2,0) nach(4,2) prüft(3,1).

## Angriffe und Flying General

Allgemeine Attack Detection prüft gegnerische geometrische Bedrohungen auf den General, ohne rekursiv legal_moves für beide Seiten aufzurufen. Das vermeidet Zyklen und macht gefesselte Angreifer-Semantik explizit: als General-Sicherheitsdefinition gelten geometrische Angriffe, unabhängig davon, ob der Angreifer dadurch seinen eigenen General öffnen würde. Diese Casual-Definition wird mit Pin-Fixtures eingefroren.

Stehen Generäle auf gleicher Datei ohne Zwischenfigur, ist die Position illegal. Ein Zug, der den letzten Blocker entfernt, ist verboten. Diese Regel gilt auch bei sonst ruhigen Zügen oder während Schachabwehr. Eine Engine darf keine Flying-General-„Capture“-Aktion als normales Fortsetzen einer bereits illegalen Position anbieten.

## Aufwand und Fehler

Ein Linienlauf ist O(max(9,10)); Kandidatenprüfung mit kompletten Probe-Boards ist bei90 Punkten gut verständlich. Erst Profiling rechtfertigt Bitboards oder inkrementelle Attack Maps. Häufige Fehler: Fluss am Ausgang statt Ziel für Elephant, Screenfarbe beschränken, Horse wie westlichen Knight behandeln, Soldier nach seitlichem Zug rückwärts lassen. Jede Tabellenzeile bekommt positive und negative Spiegeltests. [Fixtures](rule-test-positions.md) konkretisieren.
