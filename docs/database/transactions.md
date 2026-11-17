# Transaktionen und Commit-Grenze

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Angenommener Zug

Ein Kandidat entsteht im Actor aus VersionV. Innerhalb genau einer PostgreSQL-Transaktion: aktuelle Gameversion gegenV schützen (bedingtes Update oder gesperrte Zeile mit erneuter Prüfung), Snapshot aufV+1 schreiben, ein EventV+1 anlegen, bei Zug einen Move anhängen, ein Receipt für Principal+Request erstellen. Commit vor lokaler Veröffentlichung und Ack. Kein Netzwerk-Broadcast innerhalb der offenen Transaktion.

Read Committed plus explizite Versionsprüfung und Constraints genügt für diesen Einzelaggregatpfad. Serializable ist eine Alternative, verlangt aber verstandene Retry-Fehler und löst externe Nebenwirkungen nicht automatisch. SQLx-Transaction-Lifecycle und Rollback bei Drop erklären, ohne einen abgebrochenen Client als Beweis des Rollbacks zu betrachten.

## Fehler-Matrix

| Fehlerpunkt | Dauerhafter Zustand | Vorgehen |
|---|---|---|
| Validierung vor TX | V | kein Liveupdate, Domain Error |
| sicherer Rollback | V | recovery_required; aktive Partie nach sicherer DB-Rückkehr neutral abbrechen |
| Commit erfolgreich, Ack verloren | V+1 | Retry findet Receipt, kein zweiter Zug |
| Commit erfolgreich, Broadcast defekt | V+1 | Socket trennen, Snapshot/Resync |
| Verbindung beim Commit verloren | unbekannt | Writer stoppen, Receipt+Version abgleichen; keine neue ID |
| konkurrierende Version | anderer Commit | keine Überschreibung; Integritätsalarm bei unerwartetem zweiten Writer |

Ein Timeout, Resign oder Abbruch benutzt denselben atomaren Pfad. Wenn ein neuer Command erst bei now>=deadline bearbeitet wird, wird der Timeout als Ergebnis des Pfads committed, Receipt meldet timeout_before_command. Timer ohne Clientrequest benötigt keine Clientreceipt, aber die atomare einmalige terminale Version.

## Rating und Accounts

Ratingresultate sind später eigener atomarer Use Case: Resultatversion und Pool prüfen, beide Ratingzeilen in stabiler Principal-Reihenfolge sperren, genau einen rating_applications-Eintrag erzeugen und beide Werte gemeinsam schreiben. Duplicate-Ausführung findet unique Marker und ändert nichts. Aborted/casual/unberechtigte Partien bleiben ungewertet. Account-Upgrade hängt Credentials an denselben Principal, überträgt keine Sitze anhand frei behaupteter IDs.

## Nachweise

Fault Injection vor/nach jedem Schreibschritt, nach Commit vor Ack, und während Netzwerkverlust am Commit. Prüfe Snapshot, Eventzahl, Movezahl, Receipt und Ergebnis gemeinsam. Ein Test, der nur HTTP200 prüft, beweist keine Transaktionskorrektheit.
