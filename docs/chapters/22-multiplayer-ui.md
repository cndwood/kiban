# Kapitel 22 — Multiplayer-Brett, Uhranzeige und Ergebnisse

[Dokumentationsindex](../INDEX.md) · [Projektfortschritt](../checklists/project-progress.md)

## 1. Chapter Overview

Dieses Kapitel macht aus Netzwerkzuständen ein verständliches Spiel. Zwei Godot-Clients spielen eine vollständige zeitgesteuerte Xiangqi-Partie, können aufgeben und nach einer Unterbrechung den bestätigten Zustand wiederherstellen. Die Oberfläche erklärt den Unterschied zwischen ausgewähltem Zug, gesendetem Auftrag und akzeptiertem Zug.

Der sichtbare Countdown hilft bei Entscheidungen. Nur der Server darf daraus einen Timeout oder Sieg ableiten. Ein Bildschirm mit lokal null Sekunden bleibt deshalb bis zum autoritativen Ereignis ein laufendes Spiel mit möglicher Synchronisationswarnung.

## 2. Prerequisites

- Kapitel 01–21 abgeschlossen; Brettkoordinaten aus Kapitel 19 und Resync aus Kapitel 20 sind getestet.
- Serveruhren aus Kapitel 14/15 und die Recovery-Policy aus Kapitel 17 erklären können.
- Vor der Umsetzung [Clock-Spezifikation](../architecture/chess-clocks.md) und [State Synchronization](../protocol/state-synchronization.md) lesen.
- Zwei Clientfenster und ein Testmodus mit steuerbaren Snapshots stehen bereit.

## 3. Learning Objectives

- Der Entwickler kann einen UI-Zug in Auswahl, Pending und bestätigtes Serverereignis zerlegen.
- Er kann den Countdown aus einem Snapshot interpolieren, ohne lokale Zuglegalität oder Ergebnisse zu autorisieren.
- Er kann eine Versionslücke erkennen und Interaktion bis zum Resync sperren.
- Er kann reconnect, Resignation, Timeout und Serverabbruch mit unterschiedlichen Ergebnistexten darstellen.

## 4. Concepts and Theory

### State-driven Rendering

**Definition:** Sichtbare Figuren und Status werden aus einem einzigen bestätigten Clientmodell abgeleitet. **Motivation:** Unabhängige Figuren-Nodes als Datenquelle driften nach Reconnect auseinander. **Einsatz:** Ein Snapshot ersetzt das Modell atomar, danach rendert das Brett. **Fehler:** Animationspositionen werden zu fachlichen Koordinaten zurückgerechnet. **Geeignet:** Jeder Netzwerkclient und Replay. **Vertiefung:** Godots Scene-Organisation und [Frontendarchitektur](../architecture/godot-frontend.md).

### Pending Commands

**Definition:** Pending bezeichnet einen gesendeten Auftrag ohne abschließenden Receipt. **Motivation:** Nach einem Timeout ist Erfolg unbekannt. **Einsatz:** Der UI-Zustand enthält `request_id`, Quellversion und Auswahl; erneutes Senden verwendet dieselbe ID. **Fehler:** Ein neuer Snapshot wird als Beweis interpretiert, dass ein alter Auftrag nicht committed wurde. **Geeignet:** Jede Mutation über unzuverlässige Verbindungen. **Vertiefung:** [Reconnection](../architecture/reconnection.md).

### Presentational Clock Interpolation

**Definition:** Ein lokaler monotoner Zeitgeber verringert die vom Server zuletzt gemeldete Restzeit zur Anzeige. **Motivation:** Pro Frame übertragene Uhrwerte wären teuer und trotzdem verspätet. **Einsatz:** Nur die aktive Seite zählt optisch herunter; ein neuer Snapshot setzt den Anker. **Fehler:** Wall-clock-Änderung, Ping oder negative Anzeige erzeugen ein lokales Ergebnis. **Geeignet:** Responsives Feedback; nie zur Ergebnisentscheidung. **Vertiefung:** Kapitel 14 und Godots Time-Referenz.

### Version Gates

**Definition:** Ein Gate erlaubt Interaktion nur auf einem zusammenhängenden bestätigten Versionsstand. **Motivation:** Ein übersprungenes Ereignis kann Spielerfarbe, Status oder Zugrecht verändern. **Einsatz:** Ein Sprung von Version 8 auf 10 sperrt Eingabe und fordert Snapshot an. **Fehler:** Nur Figurenstand wird aktualisiert, Pending und Uhranker bleiben alt. **Geeignet:** Alle zustandsbehafteten WebSocket-Clients. **Vertiefung:** [Versionierung](../protocol/versioning.md).

## 5. Architectural Context

`GameScreen` verbindet `BoardView`, `ClockView` und `ResultPanel` mit einem `GameViewModel`. Nur der Network-Service nimmt Wire-Nachrichten entgegen. Der Server-Actor besitzt Spielstand und Uhr; das Clientmodell besitzt ausschließlich eine empfangene Projektion.

Die Eingabe benutzt die kanonischen Koordinaten x=0..8, y=0..9 aus roter Perspektive. Eine gedrehte Ansicht für Schwarz verändert ausschließlich die Abbildung auf Pixel. UI-Animationen konsumieren bereits bestätigte Übergänge. Sie dürfen das Modell weder zurücksetzen noch umsortieren.

## 6. Design Decisions

**Verbindlich:** Das MVP bewegt Figuren erst nach Serverbestätigung. Optimistic Rendering kann Verzögerung verbergen, benötigt aber Rollback, Reconciliation und verständliche Fehlerrücknahme. Für das Lernziel ist eine Pending-Markierung günstiger; bei messbar störender Latenz später neu bewerten.

**Verbindlich:** Der Server setzt Timeout am Beginn der Actor-Verarbeitung bei `now >= deadline`; Netzankunft und lokale Anzeige begründen keinen Vorrang. Diese Policy ist reproduzierbar, berücksichtigt aber Queue- und Netzlatenz nicht als Entschädigung. UI-Hinweise müssen diese Grenze ehrlich darstellen.

**Optional:** Figurenanimationen und Sound. Sie müssen bei Snapshotwechsel abbrechbar sein und dürfen den bestätigten Zustand nicht verzögern.

## 7. Implementation Tasks

- [ ] 1. Definiere im Game-View-Model bestätigten Snapshot, ausgewählte Quelle und Pending-Auftrag getrennt. Nutze eine statische Partie aus Kapitel 19; prüfe, dass Auswahl keine Brettmutation erzeugt.
- [ ] 2. Verbinde BoardView mit Spielerfarbe und aktivem Sitz aus dem Snapshot. Ein schwarzer Spieler sieht gedrehte Koordinaten; derselbe Klick muss denselben kanonischen Zielpunkt liefern.
- [ ] 3. Erlaube eine Zugauswahl nur bei aktiver Verbindung, aktuellem Snapshot, eigenem Zug und leerem Pending-Slot. Prüfe jede Sperrbedingung separat mit Fixtures.
- [ ] 4. Erzeuge bei einem bestätigten Klick einen `make_move`-Auftrag mit stabiler `request_id` und `expected_version`. Markiere Pending sichtbar, ohne die Figur im bestätigten Brett zu versetzen.
- [ ] 5. Verarbeite akzeptierte Receipts und `game_event` unabhängig von ihrer Ankunftsreihenfolge. Korrelation beendet Pending; ausschließlich aktuelle Projektionen verändern das Brett.
- [ ] 6. Zeige illegale Züge und Versionskonflikte als unterschiedliche Fälle. Bei Legalitätsfehler Quelle korrigierbar lassen; bei Versionskonflikt Snapshot anfordern und Eingabe sperren.
- [ ] 7. Ergänze ClockView mit Restzeit und lokalem monotonem Empfangsanker. Nur die aktive Uhr interpoliert; teste Pause der Anwendung und erneute Snapshotanwendung.
- [ ] 8. Klemme die sichtbare Restzeit bei null und kennzeichne fehlende Bestätigung. Starte weder Ergebnisdialog noch Zugrechtwechsel aus einem lokalen Timer; prüfe einen verzögerten Timeout-Event.
- [ ] 9. Führe beim Reconnect den vorhandenen Snapshotfluss aus. Setze Auswahl und Animationen zurück, erhalte die Identität und kläre Pending durch Receipt oder identischen Retry.
- [ ] 10. Ergänze Resignation mit bewusster Bestätigung. Das `resign`-Kommando verwendet denselben korrelierten Auftragspfad; ein Doppelklick darf kein zweites Ergebnis erzeugen.
- [ ] 11. Zeige Ergebnisgrund, Gewinner und Endzustand aus dem Servermodell. Unterscheide Checkmate, Stalemate-Verlust, Timeout, Resignation, Draw und `aborted(server_restart)`.
- [ ] 12. Halte den letzten bestätigten Stand bei Transportverlust sichtbar und zeige Verbindungsstatus. Erkläre, dass die Serveruhr weiterläuft; prüfe Reconnect kurz vor Deadline.
- [ ] 13. Beende beim Verlassen des Spiels UI-Abonnements und Animationen. Das Backendspiel läuft weiter; ein erneuter Einstieg muss denselben Snapshot statt einer neuen Partie laden.
- [ ] 14. Spiele manuell eine 3+2-Partie mit zwei Clients bis zum Ergebnis. Erzwinge darin mindestens einen illegalen Zug, einen Reconnect und einen verlorenen Response; dokumentiere die bestätigten Versionen.

## 8. Public Interfaces and Contracts

| Schnittstelle | Verantwortung | Invariante |
|---|---|---|
| `GameViewModel.apply_snapshot` | zusammengehörige Projektion, Uhranker, Status übernehmen | keine Mischung aus zwei `state_version`-Werten |
| `BoardView.move_selected` | kanonisches Quell-/Zielfeld signalisieren | kein Zug wird lokal als legal autorisiert |
| `make_move` | bestehendes WS-Kommando mit `request_id`, `game_id`, `expected_version` | Empfänger ist authentifizierter Sitz |
| `resign` | Aufgabe des eigenen aktiven Sitzes anfordern | Ergebnis kommt vom Server |
| `ClockView` | Restzeitdarstellung interpolieren | keine Ergebnis- oder Zugrechtmutation |
| `ResultPanel` | Outcome und Grund anzeigen | aborted ist kein Draw und erzeugt kein Rating |

Das Wire-Format bleibt [WebSocket Messages](../protocol/websocket-messages.md). Der Client darf zusätzliche lokale Statusfelder besitzen, überträgt sie aber nicht als Spielwahrheit.

## 9. Edge Cases

- Ack trifft vor dem zugehörigen Event ein: Pending kann enden, Brett wartet auf Projektion.
- Event trifft vor Ack ein: Brett aktualisieren, keine zweite lokale Bewegung beim Ack.
- Snapshot zeigt abgeschlossene Partie, ein alter Pending-Zug bleibt lokal: Auftrag korrelieren oder als überholt markieren, nie neu auf anderes Spiel anwenden.
- Fenster wird im Hintergrund pausiert: neuer Uhranker beim nächsten Snapshot, kein Aufsummieren verpasster Frames.
- Eine Animation endet nach einem neueren Snapshot: alte Completion darf das Modell nicht ändern.
- Serverneustart: sichtbarer Abbruch statt vermeintlicher Zeitfortsetzung.
- UI zählt null, Server akzeptiert bereits vorher verarbeiteten Zug: bestätigten Serverstand anzeigen und den Anker korrigieren.

## 10. Testing Requirements

| Given | When | Then |
|---|---|---|
| Eigener Zug auf Version 4 | gültiges Ziel gewählt | ein Pending-Auftrag, bestätigtes Brett unverändert |
| Pending-Auftrag | Event vor Ack zugestellt | genau eine sichtbare Brettänderung, Ack verschiebt keine Figur |
| Version 4 bestätigt | Event 6 trifft ein | Eingabe gesperrt, Resync ausgelöst |
| Aktive Uhr mit 1200 ms | lokale Zeit um 1500 ms fortgesetzt | Anzeige null, noch kein lokales Ergebnis |
| Schwarze Ansicht | kanonisches Feld (0,0) selektiert | korrekte gedrehte Darstellung und unveränderte Wire-Koordinate |
| Verbindungsverlust bei laufender Uhr | Reconnect gelingt | serverseitige Restzeit ersetzt Schätzung, kein Pausebonus |
| Neustart während aktiver Partie | Recovery-Snapshot kommt | `aborted(server_restart)` und gesperrte Eingabe |
| Abgeschlossene Partie | verspäteter alter Zug-Response | Ergebnis bleibt unverändert |

Steuere Response- und Eventreihenfolge mit dem Fake-Service. Uhrtests verwenden eine virtuelle lokale Zeitquelle, niemals reale Wartezeiten.

## 11. Definition of Done

- [ ] Zwei Clients können eine Partie inklusive Aufgabe und Timeout darstellen.
- [ ] Pending, Snapshot, Ergebnis und Uhranker sind als getrennte Verantwortlichkeiten umgesetzt.
- [ ] Keine lokale Uhr oder Figurenszene entscheidet fachliche Regeln.
- [ ] Versionslücken, doppelte Events und beide Ack-Reihenfolgen sind getestet.
- [ ] Reconnect und Serverabbruch besitzen nachvollziehbare UI-Texte.
- [ ] Das M5-Szenario ist mit anonymen Sessions und 3+2 nachgewiesen.

## 12. Common Pitfalls

Ein gedrehtes Brett ohne inverse Eingabeabbildung sendet gültig formatierte, aber falsche Züge. Eine pro Frame reduzierte Fließkommazahl akkumuliert Fehler und reagiert falsch auf Hintergrundpausen. Der lokale Uhranker muss aus einer monotonen Quelle stammen.

Ein Server-Ack ist kein zusätzlicher Spielzug. Wenn Ack und Event beide dieselbe Animation auslösen, erscheint ein Duplikat trotz korrektem Backend. Trenne Auftragsabschluss von Zustandsprojektion.

## 13. Reflection Questions

1. Welche Zustandsänderungen darf ein Ack auslösen, wenn der Event noch fehlt?
2. Warum ist ein lokal angezeigter Timeout kein belastbarer Spielausgang?
3. Was muss beim atomaren Snapshottausch außer den Figuren ersetzt werden?
4. Wie verändert eine Brettrotation die Anzeige, ohne das Protokoll zu verändern?
5. Welche zusätzlichen Probleme müsste Optimistic Rendering lösen?
6. Woran erkennt die UI, dass ein alter Auftrag nicht erneut mit einer neuen ID gesendet werden darf?

## 14. Further Reading

- [Godot: Time](https://docs.godotengine.org/en/stable/classes/class_time.html): monotone Tickmethoden von Systemzeit unterscheiden.
- [Godot: WebSocketPeer](https://docs.godotengine.org/en/stable/classes/class_websocketpeer.html): Polling und Verbindungszustände mit dem Service aus Kapitel 20 vergleichen; offizielle Seite am 07.11.2026 abgerufen.
- [Godot: AnimationPlayer](https://docs.godotengine.org/en/stable/classes/class_animationplayer.html): Abbruch und Lebensdauer kosmetischer Animationen untersuchen.
- [Clock-Tests](../testing/clock-tests.md): dieselben Deadlinegrenzen in Backendtests und UI-Fixtures verwenden.

## 15. Completion Record

- Abschlussdatum: ____________________
- Getroffene Designentscheidungen und Begründungen: ____________________
- Aufgetretene Schwierigkeiten und ihre Ursachen: ____________________
- Durchgeführte Tests und überprüfbare Ergebnisse: ____________________
- Verbleibende technische Schulden mit Priorität: ____________________
- Gelernte Zusammenhänge und nächster Lernschritt: ____________________
