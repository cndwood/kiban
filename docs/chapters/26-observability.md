# Kapitel 26 — Logs, Metriken, Health und Graceful Shutdown

[Dokumentationsindex](../INDEX.md) · [Projektfortschritt](../checklists/project-progress.md)

## 1. Chapter Overview

Du machst den laufenden Dienst beobachtbar und kontrolliert beendbar. Das Ergebnis sind strukturierte Logs, wenige aussagekräftige Metriken, getrennte Healthsignale und ein geprobter Shutdownablauf. Ein Betreiber kann einen festhängenden Auftrag von einem fehlenden Client unterscheiden, ohne Sessiontokens oder geheime Spielzustände auszulesen.

Observability muss Fragen beantworten: Wurde der Auftrag committed? Wie lange wartete er in der Queue? War die Datenbank verfügbar? Sie ist keine Aufforderung, alle Payloads dauerhaft zu speichern.

## 2. Prerequisites

- Kapitel 01–25 abgeschlossen; Deployment und Restore sind geprobt.
- Tokio-Tasklebensdauer, Cancellation und Commitgrenzen aus Kapitel 13/17 verstehen.
- Vorab [Backend](../architecture/backend.md), [Deployment](../architecture/deployment.md) und [Recovery](../database/recovery.md) lesen.
- Stagingzugriff und eine lokale Last-/Fehlerharness aus Kapitel 23 sind verfügbar.

## 3. Learning Objectives

- Der Entwickler kann einen Auftrag über Transport, Actor und Commit anhand korrelierter IDs verfolgen.
- Er kann hohe Metrikkardinalität erkennen und vermeiden.
- Er kann Liveness und Readiness anhand konkreter Ausfälle unterscheiden.
- Er kann einen Dienst mit laufenden Spielen innerhalb eines begrenzten Zeitbudgets beenden und nach Restart konsistente Ergebnisse zeigen.

## 4. Concepts and Theory

### Structured Logs und Spans

**Definition:** Strukturierte Logs besitzen benannte Felder; ein Span beschreibt einen zusammengehörigen Verarbeitungskontext. **Motivation:** Freitext allein lässt parallele Requests schwer zuordnen. **Einsatz:** `request_id`, `game_id`, `state_version`, Phase und Dauer verbinden Verarbeitungsschritte. **Fehler:** kompletten Command samt Token oder Junqi-Rängen als Debugfeld anhängen. **Geeignet:** Diagnose einzelner Vorgänge. **Vertiefung:** tracing-Dokumentation und OWASP Logging.

### Metrics und Cardinality

**Definition:** Eine Metrik aggregiert Messwerte; Cardinality ist die Zahl unterschiedlicher Labelkombinationen. **Motivation:** Eine Zeitreihe pro Spiel oder Request wächst unbegrenzt. **Einsatz:** Zähler für Fehlerklassen, Histogramme für Queue-/Commitlatenz und Gauges für aktive Actors. **Fehler:** `game_id` als Prometheus-Label. **Geeignet:** Trends und Alerts, nicht Einzelauftragsdiagnose. **Vertiefung:** Prometheus Instrumentation Practices.

### Liveness und Readiness

**Definition:** Liveness fragt, ob der Prozess grundsätzlich arbeitet; Readiness, ob er neue Arbeit sicher annehmen kann. **Motivation:** Ein DB-Ausfall rechtfertigt keine endlose Restartschleife, aber das Backend darf keine neuen Spiele bestätigen. **Einsatz:** Lebendiger Eventloop und startabgeschlossene, nicht drainende Instanz mit begrenztem Dependencycheck. **Fehler:** Bei jeder temporären DB-Störung den Prozess als tot markieren. **Geeignet:** Betrieb und Releaseprüfung. **Vertiefung:** [Deploymentarchitektur](../architecture/deployment.md).

### Graceful Shutdown und Cancellation

**Definition:** Shutdown beendet die Annahme neuer Arbeit, lässt begrenzte laufende Arbeit konsistent enden und räumt Ressourcen auf. Cancellation ist eine kooperative Aufforderung an Tasks. **Motivation:** Abrupt abgebrochene Writes können einen unbekannten Commitstatus hinterlassen. **Einsatz:** Readiness aus, Annahme sperren, Actorbarrieren und Commitabschluss abwarten, Sockets schließen, Ressourcen freigeben. **Fehler:** Cancellation mitten in einer Transaktion als gesicherten Rollback ausgeben. **Geeignet:** Deployments und Betriebssignale. **Vertiefung:** Tokio Graceful Shutdown.

## 5. Architectural Context

Transportspan → Actorqueuespan → Domainentscheidung → Commitspan → Receipt-/Broadcastspan bilden eine korrelierte Verarbeitungskette. Die Engine bleibt frei von globalen Loggingabhängigkeiten; der Adapter protokolliert Entscheidungen und Dauern mit redigierten Metadaten.

Der Supervisor besitzt die Taskhandles und das Shutdownsignal. Game Actors besitzen Spiele; der Supervisor darf deren internen Zustand nicht nebenläufig verändern. Aktive Partien werden beim kontrollierten Stoppen konsistent als `aborted(server_restart)` committed oder beim nächsten Start einmalig so recovered. Es entsteht kein zweiter terminaler Zustand.

## 6. Design Decisions

**Verbindlich:** Strukturierte Logs und kleine Metrikfläche statt verpflichtendem verteiltem Tracingstack. Im Monolithen reichen lokale Spans und ein geschützter Metrikexport oft aus. Externe Aggregation wird erst bei realem Betriebsbedarf ergänzt; sonst wächst die Infrastruktur schneller als der Lernnutzen.

**Verbindlich:** Metrik- und detaillierte Healthendpunkte intern oder geschützt. Eine öffentliche Minimal-Healthantwort verrät keine Datenbankcredentials, Versionstopologie oder Spielzahlen. Das erschwert direkte externe Diagnose etwas, begrenzt aber Informationsabfluss.

**Verbindlich:** Begrenzter Shutdown mit explizitem harten Ende. Unendliches Warten blockiert Releases; sofortiges Abbrechen riskiert unklare Writes. Das Zeitbudget muss größer als die vorgesehenen Commitbudgets sein und nachgemessen werden.

**Optional:** OpenTelemetry-Export nach beherrschter lokaler Korrelation; keine Microserviceannahme daraus ableiten.

## 7. Implementation Tasks

- [ ] 1. Formuliere fünf konkrete Betriebsfragen und ordne ihnen Logfelder oder Metriken zu. Vermeide Messwerte ohne Diagnosezweck; überprüfe die Liste an einem Fehlerlauf aus Kapitel 23.
- [ ] 2. Führe im Server strukturierte Spans für HTTP/WS-Auftrag und Actorverarbeitung ein. Propagiere korrelierte IDs ohne Token; ein Testauftrag muss vom Eingang bis zum Commit auffindbar sein.
- [ ] 3. Erfasse Queuewartezeit getrennt von Verarbeitung und Commitdauer. Nutze monotone Messung; ein künstlich langsames Repository darf nicht als Engine-Latenz erscheinen.
- [ ] 4. Ergänze Zähler für abgelehnte Kommandoklassen, Reconnects, Persistenzfehler und Subscriberüberlauf. Labels bleiben auf begrenzte Kategorien beschränkt; prüfe viele Spiel-IDs ohne neue Serien.
- [ ] 5. Ergänze Gauges für aktive Actors und Verbindungen sowie Histogramme für relevante Dauern. Entferne oder aktualisiere Gauges bei Taskfehler und Shutdown; wiederholte Tests dürfen keine Phantomressourcen zeigen.
- [ ] 6. Implementiere Liveness als günstige Prozessantwort ohne teure DB-Abfrage. Simuliere DB-Ausfall und prüfe, dass Liveness weiterhin die Prozessfähigkeit beschreibt.
- [ ] 7. Implementiere Readiness mit Start-/Drainstatus und begrenzter DB-Prüfung. Bei Migration, Recovery oder Shutdown muss neue Arbeit gesperrt sein; ein Timeout darf den Check nicht endlos hängen lassen.
- [ ] 8. Dokumentiere Alarme für Persistenzfehler, fehlende Readiness, knappe Platte und veraltetes Backup. Jeder Alarm enthält eine konkrete erste Diagnoseaktion statt nur einen Schwellwert.
- [ ] 9. Führe ein zentrales Shutdownsignal und eine Supervisorliste eigener Tasks ein. Erkläre, welche Tasks drainen und welche sofort aufhören dürfen; alle Handles müssen beobachtet werden.
- [ ] 10. Sperre beim Shutdown zuerst neue Sessions/Partien und neue normale Spielaufträge. Bereits angenommene Actorarbeit erhält die definierte Abschlussgrenze; teste, welcher Auftrag vor und nach der Sperre liegt.
- [ ] 11. Beende aktive Spiele über die dokumentierte terminale Transition mit derselben Commit- und Receiptpolicy. Ein parallel eintreffender Timeout oder eine Aufgabe darf kein zweites Ergebnis erzeugen.
- [ ] 12. Schließe Sockets nach finaler Projektion mit erklärtem Restartgrund, sofern der Transport noch verfügbar ist. Broadcastverlust ändert den persistierten Abschluss nicht; Reconnect muss denselben Zustand liefern.
- [ ] 13. Beende Datenbankpool und Hintergrundtasks innerhalb des Budgets. Bei Überschreitung protokolliere begrenzte Diagnosen und überlasse unklare aktive Spiele der Start-Recovery; behaupte keinen erfolgreichen Drain.
- [ ] 14. Übe SIGTERM, Taskpanik, DB-Ausfall und harten Kill im Staging. Vergleiche Logs, Metriken und gespeicherte Versionen; aktualisiere das Runbook mit messbaren Reaktionen.

## 8. Public Interfaces and Contracts

| Signal / Schnittstelle | Vertrag |
|---|---|
| `GET /health/live` | günstige Prozesslebendigkeit, keine sensiblen Details |
| `GET /health/ready` | nur ready nach abgeschlossener Recovery und außerhalb Drain; begrenzter Dependencycheck |
| Metrikexport | intern/geschützt, keine IDs oder Credentials als Labels |
| Logrecord | Zeit, Schwere, Ereignisname, begrenzte Metadaten und Korrelation; keine vollständigen geheimen Payloads |
| Shutdownsignal | einmaliger Übergang in Drain; wiederholtes Signal erzeugt keine zweite Spieltransition |
| terminales Restartresultat | `aborted(server_restart)`, kein Rating, committed vor Bestätigung |

Healthpfade sind Betriebsendpunkte außerhalb `/api/v1`. Ihre Antworten sind keine Zusage, dass jeder einzelne Auftrag erfolgreich sein wird.

## 9. Edge Cases

- Liveness gesund, DB gestört: Readiness aus und kontrollierte Fehler statt Restartsturm.
- Ein Actor panikt: Supervisor sieht den Fehler und sperrt oder recovered das Spiel; Gauge allein genügt nicht.
- SIGTERM und Deadline treffen gemeinsam ein: per-game Serialisierung entscheidet einmalig.
- DB-Commit bleibt bis zum Shutdownbudget unklar: kein Erfolg erfinden; nächste Recovery liest den tatsächlichen Stand.
- Logziel fällt aus: Spielverarbeitung darf nicht unbegrenzt auf einen Logger warten.
- Metriklabels enthalten hochvariable Fehlertexte: Speicherverbrauch kann trotz kleinem Verkehrsvolumen wachsen.

## 10. Testing Requirements

| Given | When | Then |
|---|---|---|
| Bekannter Request | Commit erfolgreich | korrelierte Spans zeigen Queue-, Verarbeitungs- und Commitdauer |
| Tausend verschiedene Spiel-IDs | Metriken exportiert | Serienzahl wächst nicht pro ID |
| DB nicht erreichbar | Healthchecks | Liveness bleibt getrennt, Readiness meldet nicht bereit |
| Laufende Partie | SIGTERM nach klarer Annahmegrenze | keine neue Partie; bestehende Transition endet konsistent |
| Timeout und Shutdown parallel | Actor verarbeitet Reihenfolge | genau ein terminales Ergebnis |
| Hängender Commit | Shutdownbudget endet | begrenzter Stop und dokumentierte Unklarheit, Recovery klärt tatsächlichen Stand |
| Markierter Token im Request | Fehler wird geloggt | Token fehlt vollständig im Diagnoseartefakt |
| Actor vollständig beendet | Metriken abgefragt | Actor-/Connection-Gauges stimmen mit Realität überein |

Tests benötigen virtuelle Zeit für Deadlines und ein separates reales, begrenztes Harnessbudget für Prozessbeendigung.

## 11. Definition of Done

- [ ] Kritische Aufträge sind über korrelierte redigierte Logs nachvollziehbar.
- [ ] Metriken besitzen begrenzte Labels und erklären Last sowie Fehler.
- [ ] Liveness und Readiness haben unterschiedliche geprüfte Ausfallsemantik.
- [ ] Graceful Shutdown und harter Kill sind beide mit Recovery getestet.
- [ ] Terminale Ergebnisse bleiben unter Shutdown-Races eindeutig und unrated bei Abbruch.
- [ ] Betreiber können anhand des Runbooks die dokumentierten Störungen selbst diagnostizieren.

## 12. Common Pitfalls

„Mehr Logs“ erhöht oft nur die Menge sensibler Daten. Ein kompletter Requestdump ist besonders gefährlich, weil er heute Tokens und später Junqi-Ränge enthalten kann. Redaction muss vor dem Logging erfolgen.

Ein Shutdownsignal beendet Tokio-Tasks nicht automatisch. Unbeobachtete Tasks und noch wartende Sender können den Prozess offenhalten. Umgekehrt ist das sofortige Droppen aller Handles kein Beweis, dass Transaktionen sauber abgeschlossen wurden.

## 13. Reflection Questions

1. Welche Diagnose wird möglich, wenn Queuewartezeit und Commitdauer getrennt gemessen werden?
2. Warum sind Spiel-IDs gute Logfelder, aber schlechte Metriklabels?
3. Wann darf ein bereiter Prozess später wieder not ready werden?
4. Was beweist ein gesundes Livenesssignal während eines Datenbankausfalls?
5. Welche Verantwortung hat der Supervisor gegenüber einem panikenden Actor?
6. Warum braucht ein begrenzter Shutdown weiterhin eine Start-Recovery?

## 14. Further Reading

- [tracing](https://docs.rs/tracing/latest/tracing/): Spans, Events und Felder der fixierten Version studieren.
- [Tokio: Graceful Shutdown](https://tokio.rs/tokio/topics/shutdown): Erkennung, Weitergabe und begrenztes Warten auf Taskabschluss auseinanderhalten.
- [Prometheus: Instrumentation](https://prometheus.io/docs/practices/instrumentation/): Metriktypen und sparsame Labels für die eigenen fünf Betriebsfragen auswählen.
- [OWASP Logging](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html): redigierte Ereignisse und Schutz der Diagnoseartefakte prüfen.
- [Recoveryplan](../database/recovery.md): Shutdownbeobachtung gegen tatsächliche Commitzustände abgleichen.

## 15. Completion Record

- Abschlussdatum: ____________________
- Getroffene Designentscheidungen und Begründungen: ____________________
- Aufgetretene Schwierigkeiten und ihre Ursachen: ____________________
- Durchgeführte Tests und überprüfbare Ergebnisse: ____________________
- Verbleibende technische Schulden mit Priorität: ____________________
- Gelernte Zusammenhänge und nächster Lernschritt: ____________________
