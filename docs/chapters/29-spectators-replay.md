# Kapitel 29 — Zuschauer, Historie und Replay

[Dokumentationsindex](../INDEX.md) · [Projektfortschritt](../checklists/project-progress.md)

## 1. Chapter Overview

Du öffnest ausgewählte Partien für lesenden Zugriff und machst vergangene Züge nachvollziehbar. Das Ergebnis sind autorisierte Zuschauerabonnements, paginierte Historie und ein Replay, dessen Zeitachse reproduzierbare bestätigte Stellungen zeigt. Dieselben Projektionsgrenzen bereiten verdeckte Spiele im nächsten Kapitel vor.

Ein Zuschauer ist eine Rolle, keine Spielerfarbe. Replay ist ein Lesemodell und führt keine Livekommandos aus. Eine abgeschlossene Partie wird durch Betrachtung nicht erneut aktiviert.

## 2. Prerequisites

- Kapitel 01–28 abgeschlossen; öffentliche Profile, sichere Projektionen und geordnete WS-Ereignisse vorhanden.
- Den Unterschied zwischen persistiertem Event, Snapshot und Frontendzustand aus Kapitel 17 erklären können.
- [State Synchronization](../protocol/state-synchronization.md), [Persistence](../architecture/persistence.md) und [Multi-Game](../architecture/multigame.md) lesen.
- Mindestens eine abgeschlossene Partie mit bekannter Zugfolge und eine private Partie als Testfixtures bereitstellen.

## 3. Learning Objectives

- Der Entwickler kann Spieler-, Zuschauer- und Historieberechtigungen explizit unterscheiden.
- Er kann einen konsistenten Zuschauerstart ohne Snapshot-/Eventlücke demonstrieren.
- Er kann `ply` und `state_version` anhand einer Partie mit Join, Zügen und Aufgabe erklären.
- Er kann Replaypositionen reproduzieren, ohne Clientlegalität als Spielautorität einzuführen.

## 4. Concepts and Theory

### Read Model und Projection

**Definition:** Ein Read Model ist eine zweckbezogene Sicht auf persistierte Daten; Projection wählt zulässige Felder für einen Empfänger aus. **Motivation:** Backendaggregate enthalten mehr als öffentlich sichtbare Spielinformationen. **Einsatz:** `PublicGameView` zeigt ein öffentliches Xiangqi-Brett, Ergebnis und freigegebene Profile; Tokens und private Lobbydaten fehlen. **Fehler:** dasselbe Aggregate für alle Empfänger serialisieren. **Geeignet:** Zuschauer, Historie und Junqi. **Vertiefung:** [Multi-Game-Architektur](../architecture/multigame.md).

### Atomic Snapshot Subscription

**Definition:** Ein Abonnement wird an genau einem Actor-Zustand mit Snapshot und anschließender geordneter Ereignisfolge verbunden. **Motivation:** Separates „erst Snapshot, dann abonnieren“ verliert dazwischenliegende Züge. **Einsatz:** Actor registriert Subscriber und liefert Projektion derselben Versionsgrenze. **Fehler:** REST-Snapshot mit unabhängigem Socket als lückenlosen Start behandeln. **Geeignet:** Spieler und Zuschauer gleichermaßen. **Vertiefung:** Kapitel 16 und [Synchronisation](../protocol/state-synchronization.md).

### Replay Cursor und History Integrity

**Definition:** Ein Replaycursor bezeichnet eine bestätigte Zug- oder Ereignisposition in einer unveränderlichen Historie. **Motivation:** UI-Navigation darf den Livezustand nicht verändern. **Einsatz:** `ply` zählt Züge; `state_version` zählt alle dauerhaften Transitionen und kann dadurch höher sein. **Fehler:** Version 12 als zwölften Zug interpretieren. **Geeignet:** Partienanalyse und Fehlerdiagnose. **Vertiefung:** [Datenbankschema](../database/schema.md).

### Bounded Fan-out

**Definition:** Ein Event wird mehreren Abonnenten mit begrenzten Einzelqueues zugestellt. **Motivation:** Ein langsamer Zuschauer darf keine Spielerpartie blockieren. **Einsatz:** Actor sendet nicht blockierend an begrenzte Queues; Überlauf trennt den betroffenen Abonnenten, der neu synchronisieren kann. **Fehler:** Auf jeden Zuschauer-Send unter Actorhoheit warten. **Geeignet:** Kleine bis mittlere Zuschauerzahlen im Monolithen. **Vertiefung:** Tokio Channels und Kapitel 13.

## 5. Architectural Context

Der Game Actor besitzt Livezustand und Subscriberliste. Authorization entscheidet vor Aufnahme die Rolle. Projektionen werden aus demselben bestätigten Zustand erzeugt; Zuschauer bekommen keine mutierenden Rechte. Persistente Historie wird über einen Read-Service geladen, nicht durch Wiederaktivierung des Actors.

Godot verwendet dieselbe BoardView mit einem separaten ReplayViewModel. Ein gewählter Replaycursor erzeugt keinen `make_move`-Auftrag. Bei gleichzeitiger Livebeobachtung bleiben Livekopf und gewählte Replayposition ausdrücklich getrennt; das MVP darf alternativ nur einen der beiden Modi pro Bildschirm anbieten.

## 6. Design Decisions

**Verbindlich:** Zuschauen zunächst ausschließlich bei öffentlichen Partien. Private Partien und ihre Historie bleiben den Seats zugänglich; ein Einladungscode zum Spielen ist kein allgemeiner Zuschauerpass. Explizite private Zuschauerrechte wären eine spätere Erweiterung. Das begrenzt die erste ACL-Matrix, erlaubt aber keine spontane öffentliche Freigabe privater Daten.

**Verbindlich:** Live-Xiangqi ohne Verzögerung im ersten Umfang, mit klar dokumentiertem Kollusionsrisiko. Eine Verzögerung würde Puffer, verzögerte Snapshots und eine eigene Uhrdarstellung benötigen. Bei wettbewerblichen Anforderungen diese Entscheidung neu bewerten; Tournament-Security wird nicht behauptet.

**Verbindlich:** Serverseitig rekonstruierte erlaubte Zustände beziehungsweise Projektionssnapshots im Replay. Das vermeidet einen zweiten vollständigen Regelkern in Godot und ermöglicht Junqi-Redaction. Es kostet zusätzliche Datenübertragung; kompaktere Client-Replaydaten sind nur bei gleicher Geheimhaltungsgarantie erlaubt.

**Optional:** Replaycache für häufige Zugpositionen. Erst messen; ein auf Korrektheit optimierter linearer Durchlauf ist bei wenigen hundert Zügen ausreichend.

## 7. Implementation Tasks

- [ ] 1. Erstelle eine Berechtigungsmatrix für öffentliche/private, aktive/abgeschlossene Partien und Seat/fremden Principal. Leite daraus konkrete Read-Service-Tests ab; private Einladungen erweitern keine Zuschauerrechte.
- [ ] 2. Definiere `PublicGameView` getrennt vom internen Aggregate. Prüfe alle Felder auf öffentliche Bedeutung; Session-, Einladungscode- und interne Repositorydaten dürfen strukturell fehlen.
- [ ] 3. Ergänze Zuschaueraufnahme im bestehenden `subscribe_game`-Anwendungsfall. Rolle wird serverseitig bestimmt; ein Clientfeld `role=player` kann keine Seatberechtigung schaffen.
- [ ] 4. Registriere Zuschauer und Ausgangssnapshot atomar im Game Actor. Erzeuge während des Einstiegs gezielt einen Zug; der neue Zuschauer muss entweder Snapshot danach oder Snapshot davor plus Event erhalten.
- [ ] 5. Wende die bestehende begrenzte Subscriberqueue auch auf Zuschauer an. Ein langsamer Viewer darf Playerbroadcast und Deadlineverarbeitung nicht verzögern; prüfe Queueüberlauf.
- [ ] 6. Verbiete mutierende Kommandos für Nicht-Seats weiterhin in der Application. Ein Zuschauer kann trotz bekannter `game_id` weder resignen noch ziehen oder einen Sitz überschreiben.
- [ ] 7. Ergänze paginierte Historie im Read-Service. Filtere Berechtigungen vor Ausgabe und verwende eine stabile Sortierung mit eindeutigem Tiebreak; prüfe mehrere Partien mit identischer Abschlusszeit.
- [ ] 8. Definiere das Replayformat mit `game_kind`, `ruleset_id`, bestätigter Startstellung und `ply`-/Versionsbezug. Dokumentiere, welche nicht-zugbezogenen Transitionen wie Aufgabe als Endmetadaten erscheinen.
- [ ] 9. Rekonstruiere angeforderte Replaypositionen serverseitig aus validierter Historie. Prüfe Regeln anhand des gespeicherten Profils; ein späteres Engineupdate darf alte Partien nicht still unter neuen Regeln interpretieren.
- [ ] 10. Behandle beschädigte oder inkompatible History als expliziten Lesefehler. Erfinde keinen fehlenden Zug und erlaube keine Mutation der unveränderlichen Partie; bewahre Diagnose ohne Credentials auf.
- [ ] 11. Baue Godot-Replaycontrols für Anfang, vorheriger/nächster Zug und Ende. Wiederverwende BoardView ausschließlich lesend; automatisches Abspielen nutzt eine Präsentationsuhr ohne Spielresultatwirkung.
- [ ] 12. Trenne in der UI „live“ und „Replayposition“. Ein neuer Liveevent darf eine bewusst zurückgesetzte Ansicht nicht kommentarlos überschreiben; der Rückkehrbutton zeigt den Livekopf.
- [ ] 13. Ergänze Zuschauerbrowser und abgeschlossene Spielansicht. Zeige Spielerprofile, Regelprofil und Ergebnisgrund; keine öffentlichen Profile dürfen Credentials oder private Historie querverlinken.
- [ ] 14. Führe einen Mehrclienttest mit zwei Spielern, normalem und langsamem Zuschauer durch. Nach Abschluss muss Replay jede bekannte Zugposition und denselben Endstand liefern.
- [ ] 15. Formuliere bereits vor Junqi den Geheimhaltungsvertrag: Replay und Spielende geben keine zuvor unbekannten Gegneridentitäten automatisch frei. Ergänze ein synthetisches verborgenes Feld und prüfe dessen Abwesenheit in öffentlichen Antworten.

## 8. Public Interfaces and Contracts

| Schnittstelle | Vertrag |
|---|---|
| `subscribe_game` als Zuschauer | authentifizierter Principal, öffentliche Partie; atomarer Snapshotstart und geordnete `game_event`-Projektionen |
| Historyliste | autorisierter Principal, Filter und Cursor; nur lesbare Partien, begrenzte Seite |
| Replayposition | `game_id`, `ply` oder spezifizierter Cursor; erlaubte Projektion mit Regelprofil |
| `ReplayViewModel` | lesende Navigation und Abspielzustand; kein fachliches Livekommando |
| Endmetadaten | terminales Outcome, Grund, letzte bestätigte Version; keine erneute Aktivierung |
| Sichtbarkeit | serverseitige Rolle bestimmt Projektion; spätere Junqi-Projektion enthält nur zeitpunktbezogen zulässiges Wissen |

REST lädt Historie und Replay. WebSocket liefert Liveprojektionen; ein Replayseek wird nicht als Spielzug in die Live-Mailbox gestellt.

## 9. Edge Cases

- Zuschauer tritt genau während eines Zugs bei: atomare Versionsgrenze verhindert Lücken.
- Privates Spiel wird über erratene ID aufgerufen: dieselbe Authorization gilt wie bei bekannter ID.
- Ergebnis trifft während Replay zurückliegender Position ein: Endstatus anzeigen, Cursor nicht ungewollt versetzen.
- History enthält Versionen ohne Move: `ply` bleibt unverändert, Version wächst.
- Langsamer Zuschauer verpasst Queuebudget: nur seine Verbindung endet; Wiederanlauf über Snapshot.
- Neues Regelprofil ersetzt altes nicht: Replay bleibt an gespeichertes `ruleset_id` gebunden.
- Junqi-Partie ist abgeschlossen: unbekannte Ränge bleiben auch im Replay verborgen.

## 10. Testing Requirements

| Given | When | Then |
|---|---|---|
| Öffentliche aktive Partie | fremder Principal abonniert | Zuschauerprojektion ohne mutierende Rechte |
| Private Partie ohne Seat | bekannte ID wird gelesen | keine History oder Brettdaten |
| Snapshotaufnahme und Zug gleichzeitig | Einstieg serialisiert | genau lückenlose bestätigte Versionen |
| Zuschauer kennt alle Koordinaten | `make_move` gesendet | Authorizationfehler, keine Mutation |
| Ein Zuschauer liest nicht | Queue läuft über | anderer Spieler empfängt weiter, Actor bleibt reaktionsfähig |
| Partie mit Join, drei Zügen und Aufgabe | Replay auf ply3 | Endbrett nach drei Zügen; state_version muss nicht 3 sein |
| Replaycursor auf ply1 | neuer Liveevent | Cursor bleibt bewusst zurückgesetzt, Livekopf aktualisiert |
| Alte Partie mit eigenem Profil | neue Defaultregeln aktiv | Replay verwendet weiter gespeichertes Profil |
| Verborgenes synthetisches Feld | öffentliche Ansicht und Replay | Feld ist nie serialisiert |

Positionen werden gegen bekannte Board-Fixtures verglichen. Abonnement-Races werden durch Actorbarrieren geprüft, nicht durch zufällige Klickgeschwindigkeit.

## 11. Definition of Done

- [ ] Öffentliche Zuschauer und private Historyrechte entsprechen der dokumentierten Matrix.
- [ ] Snapshotstart ist lückenlos und langsame Zuschauer sind isoliert.
- [ ] History und Replay sind paginiert beziehungsweise begrenzt und regelprofilgebunden.
- [ ] Godot-Replay bleibt lesend und erzeugt keine Spielkommandos.
- [ ] Jede Testposition rekonstruiert denselben bestätigten Zustand.
- [ ] Die Projektionsgrenze ist mit einem verborgenen Feld negativ getestet und für Junqi vorbereitet.

## 12. Common Pitfalls

Eine interne Movehistorie ist nicht automatisch ein sicheres öffentliches Replay. Junqi-Commands können verborgene Figuren enthalten. Schon die Downloadantwort kann Geheimnisse verraten, auch wenn das sichtbare Brett sie verdeckt.

Ein Zuschauerfanout darf die Actorloop nicht auf Netzwerkgeschwindigkeit bremsen. Unbegrenzte Pufferspeicher verschieben das Problem in einen späteren Speicherabsturz. Die dokumentierte Grenze ist Trennung und anschließender Resync.

## 13. Reflection Questions

1. Warum ist die Rollenentscheidung beim Abonnement eine Authorizationfrage?
2. Welche Lücke entsteht bei getrenntem Snapshotladen und Abonnieren?
3. Warum sind Zugnummer und Zustandsversion unterschiedliche Größen?
4. Welche Tradeoffs hat serverseitige Replayrekonstruktion gegenüber einem zweiten Clientregelwerk?
5. Warum darf Spielende bei verdeckten Spielen keine automatische Vollfreigabe bedeuten?
6. Wie würdest du später eine Zuschauerverzögerung für Snapshots und Events konsistent umsetzen?

## 14. Further Reading

- [Tokio: Channels](https://tokio.rs/tokio/tutorial/channels): begrenzte Queues und Verhalten langsamer Empfänger auf Zuschauer übertragen.
- [Godot: Scene Organisation](https://docs.godotengine.org/en/stable/tutorials/best_practices/scene_organization.html): BoardView für live und lesendes Replay wiederverwenden.
- [PostgreSQL Indexes](https://www.postgresql.org/docs/current/indexes.html): stabile Historypagination und benötigte zusammengesetzte Indizes planen.
- [Persistenzarchitektur](../architecture/persistence.md): Eventhistorie und aktuelle Projektion auseinanderhalten.
- [Multi-Game](../architecture/multigame.md): Rollenprojektion als Sicherheitsgrenze vertiefen.

## 15. Completion Record

- Abschlussdatum: ____________________
- Getroffene Designentscheidungen und Begründungen: ____________________
- Aufgetretene Schwierigkeiten und ihre Ursachen: ____________________
- Durchgeführte Tests und überprüfbare Ergebnisse: ____________________
- Verbleibende technische Schulden mit Priorität: ____________________
- Gelernte Zusammenhänge und nächster Lernschritt: ____________________
