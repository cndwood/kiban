# Junqi: veröffentlichtes eigenes Profil

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Auswahl und Quellenstatus

ruleset_id=kiban-junqi-open-v1 ist eine **eigene zweispielige Lernvariante mit verborgenen Rängen**. open bezeichnet die veröffentlichte Spezifikation, nicht offene Figuren. Sie ist keine Behauptung einer offiziellen Junqi-Norm. [Luzhanqi-Anleitung](https://www.ymimports.com/pages/how-to-play-luzhanqi) beschreibt verbreitete Figuren-/Geländemechaniken; Kiban definiert im Folgenden den vollständigen abweichenden Graphen. Vier-Spieler-Teams, Schiedsrichtervarianten und offene Varianten sind spätere eigene Profile.

## Brettgraph und Deployment

Koordinaten x0..4,y0..11. Rot gehört die Hälfte y0..5, Schwarz y6..11. Spiegelung ist(x,11−y). Alle orthogonal benachbarten Zellen besitzen Straßenkanten. Rote Camps:(1,2),(3,2),(2,3),(1,4),(3,4), schwarze gespiegelt. Zusätzlich existiert eine diagonale Straßenkante zwischen diagonal benachbarten Zellen genau dann, wenn mindestens ein Endpunkt Camp ist. HQ Rot(1,0),(3,0), Schwarz(1,11),(3,11).

Railway-Linien: vollständige Reihen y1,5,6,10 und Dateien x0,x4 von y1 bis10. Railway-Kanten verbinden orthogonal benachbarte Zellen derselben dieser Linien. Es gibt keine zusätzliche Abkürzung zwischen nichtbenachbarten Zellen. Die Straßen zwischen y5 und6 bleiben nutzbar. Damit ist der Graph vollständig bestimmbar, ohne Bilddatei oder informelle Linieninterpretation.

Jede Hälfte besitzt30 Zellen, davon5 Camps, also genau25 Deploymentplätze. Anfangs Camps leer, je Platz genau eine der25 eigenen Figuren. Flag in einem eigenen HQ; das zweite HQ darf andere Figur enthalten, die dort ebenfalls unbeweglich ist. Mines nur in den hinteren zwei Reihen (Rot0/1, Schwarz10/11), Bombs nicht in Frontreihe5/6. Server validiert vollständiges Multiset, keine freien Rangbehauptungen.

| Typ | Anzahl | Stärke |
|---|---|---|
| Commander |1 | Rang1 |
| General |1 | Rang2 |
| Division |2 | Rang3 |
| Brigade |2 | Rang4 |
| Regiment |2 | Rang5 |
| Battalion |2 | Rang6 |
| Company |3 | Rang7 |
| Platoon |3 | Rang8 |
| Engineer |3 | Rang9 |
| Mine |3 | unbeweglich |
| Bomb |2 | Sonderkampf |
| Flag |1 | unbeweglich |

submit_deployment ersetzt eigenes vollständiges Deployment vor ready. ready_deployment sperrt es. Zweites Ready aktiviert atomar die Partie, Rot beginnt. Gegner sehen nur readiness, niemals Deploymentinhalte vor Start; danach Positionen, aber unbekannte Ränge. Cancel vor beidseitiger Bereitschaft endet cancelled, keine Wertung. Pilot ist ungetaktet; kein Setup-Timeout außer allgemeinem Waiting-Ablauf24h.

## Bewegung und Kampf

Mobile Figuren ziehen einen Straßenedge oder eine beliebig lange freie Railway-Verbindung. Alle außer Engineer bleiben auf einer geraden Railway-Linie; Engineer darf an angeschlossenen Railway-Knoten abbiegen, ohne belegte Zwischenzelle zu passieren. Die erste gegnerische Figur ist ein mögliches Kampfziel, nie eine Durchgangszelle. Eigene Zielzelle unzulässig. Occupant im HQ kann nie ausziehen; Mine/Flag ohnehin nicht. Besetztes Camp ist gegen Angriffe geschützt, unabhängig von Figurenart; man darf nur ein leeres Camp betreten.

Kampfpriorität: Flagcapture gewinnt sofort; greift eine Bomb das Flag an, werden beide entfernt und der Angreifer gewinnt dennoch. Bomb gegen jede andere Figur entfernt beide (auch Mine); Engineer gegen Mine entfernt Mine und überlebt; andere mobile Angreifer gegen Mine sterben und Mine bleibt; sonst kleinere Rangzahl gewinnt, gleiche Ränge entfernen beide. Flag steht in HQ, Camp-Schutz kann Flagcapture nicht indirekt widersprechen. Beide beteiligten Kampfränge werden dauerhaft öffentlich, auch wenn die Figuren entfernt werden. Stirbt ein Commander, wird zusätzlich Position seines eigenen Flags öffentlich.

## Ergebnis, Views und Tests

Nach legaler Aktion zuerst Flagcapture, dann kein legaler Zug des nächsten Spielers als Niederlage, dann nach300 Halbzügen Remis. Keine Repetitionregel, keine Uhr, kein Rating im Pilot; Resign möglich. Beim Ende werden übrige unbekannte Ränge nicht automatisch aufgedeckt. Eigene Views enthalten eigene Ränge; Gegner/Zuschauer nur ausdrücklich offengelegte Ränge und öffentliche Positionen. Öffentliche Replay-History zeigt genau damaligen Wissensstand.

Testmatrix:25er-Multiset,30−5-Plätze, Flag in falschem HQ, Mine Front, Bomb Front, Ready doppelt, Ready/Cancel-Race, Engineer-Abbiegen, normaler Rang darf nicht abbiegen, blocker stoppt Railway, Camp immun, HQ unbeweglich, Bomb/Mine-Priorität, gleiche Ränge, Commander-Flag-Reveal und300. Halbzug. Zwei intern verschiedene gegnerische verborgene Rangbelegungen müssen bei gleichem öffentlichen Verlauf identische Zuschauer-DTOs erzeugen. Projiziere auch Errors, Receipts und Logfelder; Redaction erst im Godot-Client ist unzulässig.
