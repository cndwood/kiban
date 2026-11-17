# Kiban — Dokumentationsindex

[Repository-Start](../README.md) · [Master-Auftrag](../MASTER_PROMPT.md)

Beginne mit dem [Lernleitfaden](LEARNING_GUIDE.md) und [01 — Rust Tooling, Git, Cargo Workspaces und CI](chapters/01-workspace.md). Der [Architekturvertrag](architecture/overview.md) fixiert gemeinsame Namen und Fehlerregeln. Alle Kapitel sind selbst auszuführende Aufgaben; es wurde kein Anwendungscode vorbereitet.

## Grundlagen

- [Roadmap und Abhängigkeiten](ROADMAP.md)
- [Anforderungen und Invarianten](REQUIREMENTS.md)
- [Glossar](GLOSSARY.md)
- [Lernleitfaden](LEARNING_GUIDE.md)
- [Quellen und Versionspolitik](SOURCES.md)

## Architektur

- [Game Actors](architecture/actor-model.md)
- [Identität und Sessions](architecture/authentication.md)
- [Backend und Use Cases](architecture/backend.md)
- [Autoritative Schachuhren](architecture/chess-clocks.md)
- [Concurrency und Cancellation](architecture/concurrency.md)
- [Deployment und Betrieb](architecture/deployment.md)
- [Domain Model](architecture/domain-model.md)
- [Godot-Frontend](architecture/godot-frontend.md)
- [Mehrspielplattform und Views](architecture/multigame.md)
- [Netzwerkgrenzen](architecture/networking.md)
- [Verbindlicher Architekturvertrag](architecture/overview.md)
- [Persistenzgrenze](architecture/persistence.md)
- [Wiederverbindung und Idempotency](architecture/reconnection.md)
- [Rust-Workspace und Abhängigkeiten](architecture/rust-workspace.md)
- [Sicherheitsmodell](architecture/security.md)

## Entscheidungen

- [Entscheidungsregister](decisions/README.md)
- [ADR-001 — Modularer Monolith](decisions/ADR-001-modular-monolith.md)
- [ADR-002 — Autoritativer Server](decisions/ADR-002-authoritative-server.md)
- [ADR-003 — Ein Actor pro Partie](decisions/ADR-003-actor-model.md)
- [ADR-004 — REST und JSON-WebSocket](decisions/ADR-004-communication-protocol.md)
- [ADR-005 — PostgreSQL und Commit vor Ack](decisions/ADR-005-persistence.md)
- [ADR-006 — Monotone Zeit und Deadline](decisions/ADR-006-chess-clock-design.md)
- [ADR-007 — Getrennte Spielengines](decisions/ADR-007-game-engine-separation.md)

## Protokoll

- [Protokoll v1](protocol/overview.md)
- [REST-API](protocol/rest-api.md)
- [WebSocket-Nachrichten](protocol/websocket-messages.md)
- [Fehlerkatalog](protocol/errors.md)
- [Authentifizierungsvertrag](protocol/authentication.md)
- [State Synchronization](protocol/state-synchronization.md)
- [Versionierung und Kompatibilität](protocol/versioning.md)

## Datenbank

- [Relationales Schema](database/schema.md)
- [Migrationsplan](database/migrations-plan.md)
- [Transaktionen und Commit-Grenze](database/transactions.md)
- [Crash Recovery](database/recovery.md)

## Regeln

- [Xiangqi-Regelprofil](rules/xiangqi.md)
- [Alle sieben Figuren](rules/xiangqi-movement.md)
- [Ergebnis und Endspiel](rules/xiangqi-endgame.md)
- [Repetition, Check und Chase](rules/xiangqi-repetitions.md)
- [Junqi: veröffentlichtes eigenes Profil](rules/junqi-variants.md)
- [Positionsbasierter Regeltestkatalog](rules/rule-test-positions.md)

## Tests

- [Teststrategie und Evidenz](testing/strategy.md)
- [Unit-Tests](testing/unit-tests.md)
- [Integrationstests](testing/integration-tests.md)
- [End-to-End-Tests](testing/end-to-end-tests.md)
- [Concurrency- und Backpressure-Tests](testing/concurrency-tests.md)
- [Deterministische Uhrtests](testing/clock-tests.md)
- [Sicherheitstests](testing/security-tests.md)

## Kapitel

- [01 — Rust Tooling, Git, Cargo Workspaces und CI](chapters/01-workspace.md)
- [02 — Requirements, Domain Invariants und State Machines](chapters/02-requirements.md)
- [03 — Domain Modeling, Ownership, Enums und Errors](chapters/03-domain-modeling.md)
- [04 — Xiangqi-Board und Koordinatensysteme](chapters/04-board-representation.md)
- [05 — Figurenbewegung und Pseudo-Legal Moves](chapters/05-piece-movement.md)
- [06 — Attack Detection, Check und Legal Moves](chapters/06-legal-moves.md)
- [07 — Endgame, Repetition und Regelprofile](chapters/07-endgame-rules.md)
- [08 — Engine API, Property-Based Testing und Terminalspiel](chapters/08-engine-api.md)
- [09 — Axum, HTTP, Routing und Configuration](chapters/09-http-server.md)
- [10 — Guest Identity, Sessions und Authorization](chapters/10-guest-authentication.md)
- [Kapitel 11 — Spielanlage, Lobby und Einladungen](chapters/11-lobby-management.md)
- [Kapitel 12 — WebSocket-Transport und Anwendungsprotokoll](chapters/12-websocket-protocol.md)
- [Kapitel 13 — Game-Actors, Nebenläufigkeit und autoritativer Zustand](chapters/13-multiplayer-state.md)
- [Kapitel 14 — Schachuhrmodell, monotone Zeit und Fischer Increment](chapters/14-chess-clock-engine.md)
- [Kapitel 15 — Deadlines und zeitgesteuerte Multiplayer-Partien](chapters/15-timed-matches.md)
- [Kapitel 16 — Reconnect, Idempotency und State-Versionen](chapters/16-reconnection.md)
- [Kapitel 17 — PostgreSQL, Transaktionen und Crash Recovery](chapters/17-persistence.md)
- [Kapitel 18 — Godot-Szenen, Signals und UI-Architektur](chapters/18-godot-structure.md)
- [Kapitel 19 — Xiangqi-Brett, Koordinaten und Eingabe](chapters/19-board-rendering.md)
- [Kapitel 20 — HTTP, WebSocketPeer und Wiederverbindung in Godot](chapters/20-godot-networking.md)
- [Kapitel 21 — Lobby UI, Spielbrowser und Einladungen](chapters/21-lobby-ui.md)
- [Kapitel 22 — Multiplayer-Brett, Uhranzeige und Ergebnisse](chapters/22-multiplayer-ui.md)
- [Kapitel 23 — Integrationstests, End-to-End und Fault Injection](chapters/23-integration-testing.md)
- [Kapitel 24 — Threat Modeling, Rate Limiting und Missbrauchsschutz](chapters/24-security.md)
- [Kapitel 25 — Docker, TLS, Deployment und Backups](chapters/25-deployment.md)
- [Kapitel 26 — Logs, Metriken, Health und Graceful Shutdown](chapters/26-observability.md)
- [Kapitel 27 — Registrierte Konten, Profile und Gastübernahme](chapters/27-accounts.md)
- [Kapitel 28 — Elo, Ratingtransaktionen und Matchmaking](chapters/28-ratings-matchmaking.md)
- [Kapitel 29 — Zuschauer, Historie und Replay](chapters/29-spectators-replay.md)
- [Kapitel 30 — Junqi, Hidden Information und Multi-Game-Grenzen](chapters/30-junqi.md)

## Meilensteine

- [M1 — Fundament und Rust-Brettmodell](milestones/M1-foundation.md)
- [M2 — Lokal vollständig spielbare Xiangqi-Engine](milestones/M2-xiangqi-engine.md)
- [M3 — Zwei entfernte Terminalspieler](milestones/M3-multiplayer.md)
- [M4 — Zeit, Reconnect und Persistenz](milestones/M4-timed-games.md)
- [M5 — Vollständig spielbarer Godot-Gast-MVP](milestones/M5-godot-mvp.md)
- [M6 — Sicherer Betrieb und Beobachtbarkeit](milestones/M6-production.md)
- [M7 — Accounts, Elo, Zuschauer und Junqi](milestones/M7-expansion.md)

## Checklisten

- [Projektfortschritt](checklists/project-progress.md)
- [Quality Gates](checklists/quality-gates.md)
- [Release-Checkliste](checklists/release-checklist.md)

## Validierung und Fortsetzung

[Validierungsbericht](VALIDATION.md) enthält überprüften Umfang, Ergebnisse und Grenzen. [Restarbeiten](WORK_REMAINING.md) hält den genauen Fortsetzungsstand fest. Die dort dokumentierte Dokumentationsfertigstellung ist von den noch offenen Implementierungschecklisten zu unterscheiden.
