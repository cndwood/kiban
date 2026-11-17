# Protokoll v1

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Verantwortlichkeiten und Envelope

HTTP unter /api/v1 verwaltet Sessions, Lobby und History. /api/v1/ws ist ein authentifizierter Livekanal. Alle Anwendungsnachrichten sind UTF-8 JSON-Objekte, protocol_version ist Integer1. IDs sind opake servergenerierte Strings; Beispiele sind symbolisch, keine Validierungsregex. Versionen sind nichtnegative Integer bis2^53−1, damit Godot/JSON-Zahlen keine stillen Rundungen verursachen; Überlauf beendet Annahme und verlangt Migration.

Mutationen enthalten request_id, game_id, expected_version, type und payload. request_id wird vom Client zufällig erzeugt und für Retry unverändert gehalten. Der Server prüft Größe, Form, Session, Receipt und dann aktuelle Rechte/Version. Responses enthalten dieselbe request_id, game_id und resultierende state_version. Snapshot/resync ist keine Mutation und erhöht die Version nicht.

## Grundbeispiele

```json
{"protocol_version":1,"type":"make_move","request_id":"req-A","game_id":"game-A","expected_version":7,"payload":{"from":{"x":0,"y":3},"to":{"x":0,"y":4}}}
```

```json
{"protocol_version":1,"type":"command_ack","request_id":"req-A","game_id":"game-A","state_version":8,"payload":{"accepted":true,"outcome":"command_applied"}}
```

```json
{"protocol_version":1,"type":"error","request_id":"req-B","game_id":"game-A","error":{"code":"version_conflict","message":"Stand aktualisieren","retryable":true,"current_version":8}}
```

Nicht jedes abgewiesene Command bekommt ein dauerhaftes Receipt. Dauerhafte Receipts gelten für akzeptierte Mutationen einschließlich terminalem Übergang; eine abgelehnte Legalitäts-/Versionsprüfung bleibt ohne fachliche Wirkung. Wiederverwendung derselben akzeptierten ID mit anderem Payload wird immer abgelehnt. Nach unbekanntem Ausgang niemals neue ID erzeugen.

## Grenzen und Wartung

16KiB eingehende Commands,64KiB ausgehender Snapshot/Event, maximal50 Listeneinträge, Authentication binnen5s. Schemafehler liefern invalid_message; wiederholte Verletzung schließt Verbindung. Keine Stacktraces oder ungefilterten serde-Fehler. [Versionsregeln](versioning.md), [Fehler](errors.md), [State Sync](state-synchronization.md) sind verbindlich.

command_ack.payload enthält accepted=true und outcome=command_applied oder timeout_before_command. Im zweiten Fall wurde nur das dauerhafte Timeout-Ergebnis angenommen, nicht der angefragte Zug. Die vollständige View kommt über Event/Snapshot; ein Ack enthält keinen geheimen Enginezustand.
