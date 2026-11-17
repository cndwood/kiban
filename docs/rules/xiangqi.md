# Xiangqi-Regelprofil

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Gewähltes Profil

ruleset_id=kiban-xiangqi-casual-v1. Bewegung, Palast, Fluss, Schach und Patt orientieren sich an Xiangqi; Repetition ist eine ausdrücklich eigene Casual-Vereinfachung. Das Profil darf nicht als WXF-turnierkonform vermarktet werden. Normative Vergleichsquelle: [World Xiangqi Rules2018, Kapitel1 und illustrierte Wiederholungsregeln](https://www.wxf-xiangqi.org/images/wxf-rules/2018_World_XiangQi_Rules_English2018.pdf). Turniermodus erfordert eine neue ruleset_id, vollständige Interpretation der Chase-Kategorien und separat überprüfte Regelpositionen.

## Brett und Start

9 Dateien x=0..8,10 Reihen y=0..9, aus roter Sicht x links→rechts und y roter Grundreihe→schwarzer Grundreihe. Figuren stehen auf Schnittpunkten. Rot zieht zuerst. Palast Rot x3..5,y0..2, Schwarz x3..5,y7..9. Fluss zwischen y4 und5. Start Rot: y0 Chariot,Horse,Elephant,Advisor,General,Advisor,Elephant,Horse,Chariot; Cannons(1,2),(7,2); Soldiers(0,3),(2,3),(4,3),(6,3),(8,3). Schwarz spiegelt y→9−y; x bleibt gleich.

## Geltung

Jeder Zug bewegt genau eine eigene Figur auf einen anderen Schnittpunkt, schlägt gegebenenfalls eine gegnerische Figur und darf den eigenen General nicht angegriffen lassen. Keine Rochade, Umwandlung, en passant oder westliche Pattregel. Der General wird nicht als normaler erfolgreicher Zug geschlagen: Partie endet durch keine legalen Antworten beziehungsweise andere Ergebnisbedingungen. Attack Detection darf gegnerische Generalbedrohung modellieren, ohne einen Spielzustand ohne General zu akzeptieren.

Das Profil hat kein automatisches insufficient-material-Remis, keine50-Züge-Regel und kein eingebautes Remisangebot im MVP. Damit manche ruhigen Partien nicht automatisch enden, ist bewusst; Aufgabe und Zeitlimit bleiben verfügbar. Perpetual Chase wird nicht automatisch als Verstoß klassifiziert. [Repetition](xiangqi-repetitions.md) definiert das vereinfachte Ende präzise.

## Abnahme

Jede Partie speichert ruleset_id unveränderlich. UI zeigt Casual-Hinweis; Historie zeigt das ursprüngliche Profil. Eine spätere Änderung wirkt nur auf neue Partien. [Bewegung](xiangqi-movement.md), [Enden](xiangqi-endgame.md) und [Testpositionen](rule-test-positions.md) sind Teil dieses Vertrags.
