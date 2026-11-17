# Kapitel 23 — Integrationstests, End-to-End und Fault Injection

[Dokumentationsindex](../INDEX.md) · [Projektfortschritt](../checklists/project-progress.md)

## 1. Chapter Overview

Du baust ein reproduzierbares Nachweissystem für die Übergänge zwischen Engine, Actor, Datenbank, Protokoll und UI. Einzelne Unit-Tests können korrekt sein, obwohl ein Zug zweimal gespeichert oder ein Fehler falsch übertragen wird. Integrationstests prüfen diese Grenzen; End-to-End-Szenarien prüfen den erlebbaren Gesamtweg.

Das Ergebnis ist ein kleiner, aussagekräftiger Satz von Testläufen, der dieselben Fehlerbedingungen lokal und in CI erzeugt. Zufällige Sleeps werden durch kontrollierbare Barrieren, Fake-Zeit und explizite Zustandsbeobachtung ersetzt.

## 2. Prerequisites

- Kapitel 01–22 abgeschlossen; Engine- und Clock-Unit-Tests existieren bereits.
- SQLx-Transaktionen, Tokio-Tasks und WebSocket-Reconnect sind verstanden.
- Lokales PostgreSQL und getrennte Testdatenbanken stehen bereit; eine Produktionsdatenbank wird niemals benutzt.
- [Teststrategie](../testing/strategy.md), [Concurrency-Tests](../testing/concurrency-tests.md) und [Recovery](../database/recovery.md) vorab lesen.

## 3. Learning Objectives

- Der Entwickler kann Unit-, Integrations-, Vertrags- und End-to-End-Test nach überprüfter Grenze unterscheiden.
- Er kann einen Crash vor und nach Commit gezielt auslösen und unterschiedliche erwartete Ergebnisse begründen.
- Er kann einen konkurrierenden Join ohne Timingzufall testen.
- Er kann einen fehlgeschlagenen Lauf anhand Seed, Ereignisfolge und redigierten IDs reproduzieren.

## 4. Concepts and Theory

### Test Boundary

**Definition:** Eine Testgrenze legt fest, welche Komponenten real und welche ersetzt sind. **Motivation:** Unklare Grenzen erzeugen falsches Vertrauen; ein Mock-Datenbanktest prüft keine SQL-Constraint. **Einsatz:** Actor plus echte Testdatenbank prüft Commit-Atomizität, HTTP plus Fake-Zeit prüft Timer ohne Godot. **Fehler:** Jeden Test End-to-End nennen oder alles ausschließlich durch die UI prüfen. **Geeignet:** Jede Testplanung. **Vertiefung:** [Teststrategie](../testing/strategy.md).

### Deterministic Scheduling

**Definition:** Relevante Fortsetzungspunkte eines nebenläufigen Tests werden bewusst kontrolliert. **Motivation:** Ein erfolgreicher Lauf nach einem Sleep beweist keine Race-Freiheit. **Einsatz:** Zwei Join-Aufträge warten an einer Barriere, bevor sie denselben letzten Sitz anfragen. **Fehler:** Eine einzige zufällige Reihenfolge als vollständigen Beweis ansehen. **Geeignet:** Actor-, Shutdown- und Persistenzübergänge. **Vertiefung:** Tokios Testzeit und Synchronisationsdokumentation.

### Fault Injection

**Definition:** Ein Fehler wird an einer benannten Systemgrenze absichtlich ausgelöst. **Motivation:** Netzabbrüche und Commitfehler sind schwer manuell zu treffen. **Einsatz:** Repository-Failpoint vor Commit; Prozessabbruch nach Commit und vor Broadcast. **Fehler:** Test-Hooks unbegrenzt in Production aktiv lassen. **Geeignet:** Recovery, Retry und Outbox-ähnliche Effekte; Kiban nutzt keine verpflichtende Messaginginfrastruktur. **Vertiefung:** [Recovery](../database/recovery.md).

### Contract Testing

**Definition:** Ein Vertragstest überprüft Wire-Form und fachliche Bedeutung einer Schnittstelle. **Motivation:** Zwei intern richtige Komponenten können unterschiedliche Fehlercodes erwarten. **Einsatz:** JSON-Fixtures prüfen Version, Feldtypen und Projektion gegen Server und Godot-Parser. **Fehler:** Nur erfolgreiche Deserialisierung prüfen und fehlende Authorization übersehen. **Geeignet:** REST/WS und regelprofilabhängige Snapshots. **Vertiefung:** [Protokollübersicht](../protocol/overview.md).

## 5. Architectural Context

Die Tests liegen später an den jeweiligen Grenzen: reine Engine-Tests ohne Tokio; Application-Tests mit Fake-Clock und Repository-Vertrag; SQLx-Integration mit PostgreSQL; Server-Vertragstests mit zwei Terminalclients; wenige Godot-End-to-End-Läufe.

Ein Testlauf besitzt Datenbanknamespace, Sessiondaten und Prozesshandles. Sein Teardown beendet eigene Tasks und löscht ausschließlich eigene Testdaten. Die Test-Harness ist ein Adapter und darf keine produktive Legalitätsentscheidung ersetzen. Die Primärinvariante lautet: ein akzeptierter Auftrag erzeugt höchstens eine persistierte Transition und höchstens ein Ergebnis.

## 6. Design Decisions

**Verbindlich:** Viele schnelle deterministische Grenztests und wenige vollständige UI-Läufe. Nur End-to-End zu testen wäre realitätsnah, aber langsam und schlecht diagnostizierbar; nur Mocks übersehen Datenbank- und Transportfehler. Die Kombination passt zur modularen Architektur. Bei neuen Fehlerklassen wird die passende Grenze erweitert.

**Verbindlich:** Eine echte PostgreSQL-Testinstanz für Transaktionen und Constraints. In-Memory-Speicher prüft keine Isolation. Das kostet Setupzeit, verhindert aber den irreführenden Ersatz relationaler Semantik durch eine Map.

**Optional:** Systematische Scheduler-Exploration für isolierbare Synchronisationsbausteine. Erst einsetzen, wenn die zu prüfende Zustandsmenge und ihr Nutzen klar begrenzt sind.

## 7. Implementation Tasks

- [ ] 1. Erstelle eine Testmatrix in der Testdokumentation: Invariante, Grenze, reale Komponenten, Fake-Komponenten und erwarteter Beweis. Weise jedem bekannten Risiko genau den günstigsten belastbaren Test zu.
- [ ] 2. Baue für Servertests eine Harness mit zufälligem freien Port und explizitem Ready-Signal. Ein Test darf erst nach bestätigter Bereitschaft Requests senden; entferne Start-Sleeps.
- [ ] 3. Isoliere SQLx-Tests durch eigene Datenbank oder eindeutig getrenntes Schema. Prüfe vor Cleanup die Testkennung; dokumentiere Schutz vor Zugriff auf produktive URLs.
- [ ] 4. Ergänze zwei unabhängig authentifizierte Terminal-Testclients. Sie müssen Snapshots, korrelierte Receipts und geordnete Events aufzeichnen können; Session-Tokens bleiben aus Logs entfernt.
- [ ] 5. Schreibe den konkurrierenden Join-Test mit Barriere am Application-Eingang. Genau ein Auftrag bekommt den letzten Sitz; der Verlierer erhält einen stabilen Konfliktcode.
- [ ] 6. Teste denselben `make_move` zweimal mit gleicher `request_id`. Verwende zunächst identischen, danach veränderten Payload; prüfe ein Event, einen Receipt und expliziten Payloadkonflikt.
- [ ] 7. Erzwinge im Repository einen Fehler vor Commit. Keine Erfolgsmeldung darf erscheinen; Actor wechselt nach Policy in `recovery_required` und nimmt keine weiteren normalen Züge an.
- [ ] 8. Beende den Server nach Commit vor Broadcast über einen kontrollierten Failpoint. Nach Restart muss der gespeicherte Zug genau einmal vorhanden und das ehemals aktive Spiel `aborted(server_restart)` sein.
- [ ] 9. Trenne einen WebSocket nach gesendetem Zug. Verbinde dieselbe Session erneut, sende dieselbe ID und prüfe, dass Receipt vor Versionskonflikt ausgewertet wird.
- [ ] 10. Prüfe Clock-Grenzen mit Fake-Zeit: eine Einheit vor Deadline, genau Deadline, danach. Jeder Test muss dieselbe Actor-Verarbeitungspolitik benutzen; Netzzeitstempel sind irrelevant.
- [ ] 11. Simuliere einen langsamen Abonnenten und fülle dessen begrenzte Queue. Der Actor darf nicht blockieren; betroffener Client wird getrennt und kann per Snapshot nachladen.
- [ ] 12. Ergänze gemeinsame JSON-Vertragsfixtures für Snapshot, Fehler, Ergebnis und unbekannte Protokollversion. Server und Godot-Decoder müssen dieselben gültigen und ungültigen Fälle behandeln.
- [ ] 13. Definiere einen kurzen Godot-End-to-End-Lauf: Gaststart, öffentliche Erstellung, Join, legaler Zug, Reconnect, Aufgabe, Historie. Bestätige Ergebnisse anhand Serverversionen statt nur Screenshotpixeln.
- [ ] 14. Integriere schnelle Tests und Datenbanktests getrennt in die bestehende CI. Bewahre bei Fehlern Seed, Versionen und redigierte Ereignisfolge auf; wiederhole keinen fehlgeschlagenen Test still bis er grün wird.
- [ ] 15. Führe alle Failure-Szenarien zweimal in unterschiedlicher Reihenfolge aus. Prüfe nach Teardown offene Ports, Tasks und Datenbankreste; dokumentiere Laufzeit und verbleibende Grenzen.

## 8. Public Interfaces and Contracts

| Testadapter | Steuerbare Eingabe | Beobachtbarer Ausgang |
|---|---|---|
| `TestClock` | definierter monotoner Fortschritt | Deadlineentscheidung ohne reale Wartezeit |
| Repository-Failpoint | Fehler vor Commit / Unterbrechung danach | kein Ack vor Commit; bestätigte Historie nach Recovery |
| Barrier am Join | zwei wartende Principals | genau ein zweiter Sitz |
| Fake-Transport | Drop, Duplicate, geänderte Reihenfolge zwischen Reconnects | Retry und Resync bleiben konsistent |
| Test-Trace | `request_id`, `game_id`, `state_version`, Seed | reproduzierbare Reihenfolge ohne Credentials |

Failpoints sind testgebundene Adapter oder explizit deaktivierte Entwicklungsfunktionen. Sie sind keine öffentlich erreichbaren administrativen HTTP-Endpunkte.

## 9. Edge Cases

- Ein Client wartet auf ein Event, das wegen Disconnect nie eintrifft: begrenzter Testtimeout mit Diagnose statt Hänger.
- Commitstatus ist nach Verbindungsfehler unbekannt: Retry über Receipt und Recovery, nicht anhand eines Mock-Rückgabewerts raten.
- Taskpanik wird von einer unbeobachteten Join-Handle verschluckt: Harness muss Taskfehler einsammeln.
- Cleanup läuft nach fehlgeschlagenem Setup: nur bereits erzeugte eigene Ressourcen anfassen.
- Ein Fixture passt zur aktuellen Implementierung, aber verletzt das veröffentlichte Protokoll: Vertrag als eigene Prüfgrundlage verwenden.
- Reale Uhr und Fake-Uhr werden gemischt: Deadline-Test wird nondeterministisch und muss als fehlerhaft gelten.

## 10. Testing Requirements

| Given | When | Then |
|---|---|---|
| Ein freier Sitz | zwei Join-Aufträge synchron freigegeben | genau ein Gewinner, keine Doppelbelegung |
| Ein gültiger Zug und Receipt | Retry mit gleicher ID nach Reconnect | gleicher Abschluss, keine weitere Version |
| Gleiche ID, anderer Zielpunkt | zweiter Auftrag kommt | Payloadkonflikt und unveränderter Spielstand |
| Persistenz-Failpoint vor Commit | legaler Zug verarbeitet | kein erfolgreicher Receipt und keine Broadcast-Bestätigung |
| Commit erfolgreich | Prozess vor Broadcast stirbt | History enthält Zug einmal; aktive Partie wird beim Start abgebrochen |
| Actor kurz vor Deadline | Fake-Zeit erreicht exakt Deadline | Timeout gewinnt vor Movevalidierung |
| Volle Subscriber-Queue | nächster Event entsteht | langsamer Client getrennt, anderer Spieler erhält Event |
| Test beendet oder fehlgeschlagen | Teardown läuft | eigene Tasks beendet, produktive Daten unberührt |

Ein Test gilt nur als bestanden, wenn auch Abwesenheit unerlaubter Effekte geprüft wird: keine zweite Zeile, kein zweites Rating, kein alter Subscriber.

## 11. Definition of Done

- [ ] Jede kritische Commit-, Retry-, Clock- und Join-Invariante besitzt einen reproduzierbaren Grenztest.
- [ ] PostgreSQL-Tests prüfen echte Constraints und Rollback.
- [ ] Mindestens ein Godot-Lauf beweist den vollständigen M5-Pfad.
- [ ] Fehlertraces enthalten keine Credentials oder später vertraulichen Junqi-Ränge.
- [ ] Harness und Teardown sind auch bei Panik oder Abbruch begrenzt.
- [ ] CI-Ergebnisse, Laufzeiten und Testgrenzen sind in der Teststrategie dokumentiert.

## 12. Common Pitfalls

Ein grüner Test, der nur auf eine Nachricht wartet, übersieht unerlaubte zweite Nachrichten. Prüfe erwartete Zähler und Endzustände. Ein zufälliger Port ohne Ready-Signal löst nicht das Start-Race.

Ein Retry in der Test-Harness kann Produktfehler verstecken. Wiederholbarkeit ist Diagnose, kein Ersatz für eine deterministische Spezifikation. Ein Prozesscrash wird außerdem nicht durch einen Fehler-Return simuliert: unvollständige Prozesse und offene Transaktionen sind eine eigene Fehlerklasse.

## 13. Reflection Questions

1. Welcher Fehler bleibt unsichtbar, wenn alle Repositorytests eine In-Memory-Map verwenden?
2. Welche zwei unterschiedlichen Ergebnisse erwartest du vor und nach Commit bei einem Prozesscrash?
3. Warum muss ein Duplicate-Test auch Event- und Zeilenzahl prüfen?
4. Wie begrenzt du einen wartenden Test, ohne Fachzeit und Testtimeout zu vermischen?
5. Welche Interleavings beweist eine Barriere und welche bleiben ungetestet?
6. Wieso sollte ein UI-End-to-End-Lauf nicht sämtliche Clock-Grenzen prüfen?

## 14. Further Reading

- [Tokio: Unit Testing](https://tokio.rs/tokio/topics/testing): pausierte Zeit und Testgrenzen gezielt durcharbeiten.
- [SQLx: Dokumentation](https://docs.rs/sqlx/latest/sqlx/): Testunterstützung, Transaktionen und Pool-Lebensdauer der tatsächlich fixierten Version nachlesen.
- [PostgreSQL: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html): konkurrierende Schreibvorgänge und Retry-Gründe verstehen.
- [Godot: Headless-Betrieb](https://docs.godotengine.org/en/stable/tutorials/editor/command_line_tutorial.html): reproduzierbare UI-Testläufe und Exitstatus planen.
- [End-to-End-Spezifikation](../testing/end-to-end-tests.md): M5-Abnahmefälle als feste Regression übernehmen.

## 15. Completion Record

- Abschlussdatum: ____________________
- Getroffene Designentscheidungen und Begründungen: ____________________
- Aufgetretene Schwierigkeiten und ihre Ursachen: ____________________
- Durchgeführte Tests und überprüfbare Ergebnisse: ____________________
- Verbleibende technische Schulden mit Priorität: ____________________
- Gelernte Zusammenhänge und nächster Lernschritt: ____________________
