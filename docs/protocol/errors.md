# Fehlerkatalog

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Einheitliche Form

error enthält code (stabiler englischer Bezeichner), message (anzeigbarer Hinweis), retryable und optional current_version oder retry_after_ms. request_id und game_id werden nur übernommen, wenn valide und bekannt. Keine internen SQL-/Engine-Debugdaten.

| Code | HTTP | Bedeutung und Clientreaktion |
|---|---|---|
| invalid_message |400 | Form, Größe oder Pflichtfelder korrigieren |
| unsupported_version |400 | kompatiblen Client verwenden |
| unauthenticated |401 | Session erneuern, nicht fremden Sitz übernehmen |
| forbidden |403 | bekannte öffentliche Ressource, Aktion nicht erlaubt |
| not_found |404 | unbekannt oder privat ohne Recht; nicht unterscheiden |
| game_full |409 | Liste aktualisieren |
| invalid_state |409 | Aktion in falscher Phase |
| version_conflict |409 | Snapshot beziehen; Nutzerabsicht neu prüfen |
| request_id_conflict |409 | akzeptierte ID mit anderem Payload; Clientfehler |
| illegal_move |422 | Brett unverändert, Auswahl korrigieren |
| rate_limited |429 | retry_after respektieren |
| busy |503 | keine Annahme; gleicher Request später |
| persistence_unavailable |503 | Ausgang ggf unklar; Receipt/Resync statt neuer ID |
| internal_error |500 | Korrelation protokollieren, keine Details zeigen |

Ein Timeout ist kein illegal_move: die Partie erhält ein dauerhaftes Ergebnis und Event, ein nachfolgender Zug erhält invalid_state. Falls der verarbeitete Command selbst die Deadline-Prüfung auslöst, ack beschreibt die angenommene terminale Systemauflösung statt den Zug als ausgeführt darzustellen; receipt result enthält outcome=timeout_before_command. UI muss den Snapshot lesen, nicht accepted=true als „Figur bewegt“ interpretieren.

## Überprüfung

Falscher Spielerzug, illegaler Zug und unbekannte/private GameId benötigen unterschiedliche interne Ursachen, aber keine unnötige Informationspreisgabe. Payload zu groß möglichst vor JSON-Allokation begrenzen. Serde-Fehler auf eigene Codes übersetzen. Alle Fehlerzustände aus Terminalclient, Godot und HTTP-Test identisch behandeln.
