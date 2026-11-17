# Backend und Use Cases

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Schichten

Axum extrahiert und begrenzt HTTP/WS-Eingaben, authentifiziert und übersetzt Errors. Application ordnet Identität und Rechte zu, routet an Lobby oder Game Actor und koordiniert Transaktionen. Die Engine beantwortet ausschließlich Regeln. SQLx-Adapter implementieren Repository-Ports. Kein Handler verändert ein Brett direkt.

Konfiguration unterscheidet bind address, public origin, DB-Verbindung, Tokenlaufzeit, Mailboxgrößen und Secrets. Fehlende Pflichtwerte verhindern Start; Debug-Ausgaben dürfen keine Zugangsdaten enthalten. Readiness benötigt initialisierte Repositories und abgeschlossene Recovery; Liveness nur einen funktionierenden Prozess, keinen erfolgreichen DB-Query.

## Ablauf Spiel erstellen

```mermaid
sequenceDiagram
  participant C as Client
  participant H as Axum
  participant L as Lobby
  participant D as Repository
  C->>H: POST games mit Session und request_id
  H->>L: authentifizierter CreateGame
  L->>D: Spiel und Create-Receipt committen
  D-->>L: Erfolg
  L-->>H: game_id und erlaubte Einladung
  H-->>C: 201 Created
```

Ab Kapitel17 müssen auch REST-Mutationen retry-sicher sein. Create-Receipts verwenden Principal+request_id+Operation als Scope, weil noch keine GameId existiert. Join/Cancel laufen im Game-Scope. Request-Abbruch darf einen begonnenen DB-Commit nicht als sicheren Rollback behandeln; Ergebnis später über Receipt abfragen.

## Lern- und Betriebsgrenzen

Zunächst ein Prozess mit festem Kapazitätslimit. CPU-lastige Analysen und unbeschränktes JSON gehören nicht auf Runtime-Tasks. Skalierung über mehrere Server ist keine einfache Load-Balancer-Änderung: Actor-Ownership müsste mit Fencing und Routing neu entworfen werden. [Concurrency](concurrency.md) definiert diese Grenze.

## Autoritativer Move-Pfad ab Kapitel17

```mermaid
sequenceDiagram
  participant C as Client
  participant T as Transport
  participant A as Game Actor
  participant E as Xiangqi Engine
  participant D as PostgreSQL
  C->>T: make_move mit Request und erwarteter Version
  T->>A: validierte Form und authentifizierter Principal
  A->>A: Receipt und aktuelle Rechte pruefen
  A->>A: monotone Deadline am Bearbeitungsbeginn pruefen
  A->>E: Bewegung und General-Sicherheit pruefen
  E-->>A: Kandidat oder Domain Error
  A->>A: Outcome und Clock-Kandidat bilden
  A->>D: Snapshot Event Move Receipt atomar committen
  D-->>A: bestaetigter Commit
  A-->>C: command_ack und berechtigte game_event View
```

Das Diagramm zeigt den neuen legalen Move vor Deadline. Duplicate Requests enden am Receipt; Fehler enden ohne Kandidatcommit. Ist Deadline erreicht, läuft statt Engine-Move der terminale Timeoutpfad durch dieselbe Persistenzgrenze. Transport, Actor und Engine sind verschiedene Verantwortlichkeiten, keine drei Dienste.
