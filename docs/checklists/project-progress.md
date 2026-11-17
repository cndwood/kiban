# Projektfortschritt

[Index](../INDEX.md) · [Quality Gates](quality-gates.md)

Alle Kästchen erfassen **zukünftige Eigenimplementierung**, nicht das Vorhandensein dieser Dokumente. Ausgangsstand: Anwendung nicht begonnen. Ein Haken benötigt Test-/Journalnachweis.

| Meilenstein | Kapitel | Abgeschlossen | Gate |
|---|---|---|---|
| [M1](../milestones/M1-foundation.md) |01–04|0/4|offen|
| [M2](../milestones/M2-xiangqi-engine.md) |05–08|0/4|offen|
| [M3](../milestones/M3-multiplayer.md) |09–13|0/5|offen|
| [M4](../milestones/M4-timed-games.md) |14–17|0/4|offen|
| [M5](../milestones/M5-godot-mvp.md) |18–22|0/5|offen|
| [M6](../milestones/M6-production.md) |23–26|0/4|offen|
| [M7](../milestones/M7-expansion.md) |27–30|0/4|offen|

## Phase I · M1 — Fundament und Rust-Brettmodell

- [ ] [Kapitel 01: Rust-Tooling, Git, Cargo Workspaces und CI](../chapters/01-workspace.md) — DoD und Completion Record vollständig.
  - [ ] Toolchain/Git/CI selbst aufsetzen.
  - [ ] Workspace-Grenzen nachweisen.
  - [ ] fmt/clippy/Tests ausführen.
- [ ] [Kapitel 02: Anforderungen und Invarianten](../chapters/02-requirements.md) — DoD und Completion Record vollständig.
  - [ ] Anforderungen priorisieren.
  - [ ] Lifecycle und Invarianten definieren.
  - [ ] Akzeptanzfälle zuordnen.
- [ ] [Kapitel 03: Domain Modeling und Rust Ownership](../chapters/03-domain-modeling.md) — DoD und Completion Record vollständig.
  - [ ] IDs und Zustände modellieren.
  - [ ] Ownership und Borrowing erklären.
  - [ ] Fehlergrenzen testen.
- [ ] [Kapitel 04: Brett und Koordinaten](../chapters/04-board-representation.md) — DoD und Completion Record vollständig.
  - [ ] 90-Punkte-Brett abbilden.
  - [ ] Startposition und Spiegelung prüfen.
  - [ ] Koordinatengrenzen testen.

## Phase II · M2 — Lokal vollständig spielbare Xiangqi-Engine

- [ ] [Kapitel 05: Figuren und Pseudo-Legalität](../chapters/05-piece-movement.md) — DoD und Completion Record vollständig.
  - [ ] Sieben Figuren separat prüfen.
  - [ ] Horse/Elephant/Cannon-Blocker testen.
  - [ ] Pseudo-Legalität spiegeln.
- [ ] [Kapitel 06: Schach und legale Züge](../chapters/06-legal-moves.md) — DoD und Completion Record vollständig.
  - [ ] Attack Detection ohne Rekursion.
  - [ ] Probezüge und Flying General.
  - [ ] General-Sicherheit nachweisen.
- [ ] [Kapitel 07: Endspiel und Wiederholungen](../chapters/07-endgame-rules.md) — DoD und Completion Record vollständig.
  - [ ] Matt/Patt unterscheiden.
  - [ ] Casual-Repetition implementieren.
  - [ ] Turniergrenzen kennzeichnen.
- [ ] [Kapitel 08: Engine-API und lokale Partie](../chapters/08-engine-api.md) — DoD und Completion Record vollständig.
  - [ ] Engine-Vertrag stabilisieren.
  - [ ] Properties und Regressionen.
  - [ ] Terminalpartie demonstrieren.

## Phase III · M3 — Zwei entfernte Terminalspieler

- [ ] [Kapitel 09: Axum und HTTP](../chapters/09-http-server.md) — DoD und Completion Record vollständig.
  - [ ] Axum-Routen und Config.
  - [ ] HTTP-Error-Mapping.
  - [ ] Transport ohne Engine-IO testen.
- [ ] [Kapitel 10: Gastidentität und Sessions](../chapters/10-guest-authentication.md) — DoD und Completion Record vollständig.
  - [ ] Token erzeugen und hashen.
  - [ ] Sessionablauf/Widerruf.
  - [ ] Sitzautorisierung testen.
- [ ] [Kapitel 11: Lobby und Einladungen](../chapters/11-lobby-management.md) — DoD und Completion Record vollständig.
  - [ ] Create/List/Join/Invite.
  - [ ] Parallele Sitzvergabe.
  - [ ] Private Liste verhindern.
- [ ] [Kapitel 12: WebSocket-Protokoll](../chapters/12-websocket-protocol.md) — DoD und Completion Record vollständig.
  - [ ] Envelope und Lifecycle.
  - [ ] Snapshots/Acks/Errors.
  - [ ] Frames und Limits prüfen.
- [ ] [Kapitel 13: Game Actors und Synchronisation](../chapters/13-multiplayer-state.md) — DoD und Completion Record vollständig.
  - [ ] Actor-Ownership.
  - [ ] Mailbox/Backpressure.
  - [ ] Version- und Join-Races.

## Phase IV · M4 — Zeit, Reconnect und Persistenz

- [ ] [Kapitel 14: Monotone Schachuhren](../chapters/14-chess-clock-engine.md) — DoD und Completion Record vollständig.
  - [ ] Monotone Clock-Abstraktion.
  - [ ] Fischer-Budgets.
  - [ ] Fake-Time-Grenztests.
- [ ] [Kapitel 15: Deadlines und Zeitpartien](../chapters/15-timed-matches.md) — DoD und Completion Record vollständig.
  - [ ] Deadline/Move-Ordnung.
  - [ ] Timer-Generationen.
  - [ ] Timeout genau einmal.
- [ ] [Kapitel 16: Reconnect und Idempotency](../chapters/16-reconnection.md) — DoD und Completion Record vollständig.
  - [ ] Stable Request-Retry.
  - [ ] Snapshot plus Stream.
  - [ ] Gap und Ackverlust prüfen.
- [ ] [Kapitel 17: PostgreSQL und Recovery](../chapters/17-persistence.md) — DoD und Completion Record vollständig.
  - [ ] Schema/Migrationen selbst schreiben.
  - [ ] Atomarer Commit vor Ack.
  - [ ] Crash/Restore nachweisen.

## Phase V · M5 — Vollständig spielbarer Godot-Gast-MVP

- [ ] [Kapitel 18: Godot-Szenen und Services](../chapters/18-godot-structure.md) — DoD und Completion Record vollständig.
  - [ ] Scenes/Signals/Autoload.
  - [ ] UI/Netzwerk-Grenzen.
  - [ ] Statische Fixtures.
- [ ] [Kapitel 19: Brettdarstellung und Eingabe](../chapters/19-board-rendering.md) — DoD und Completion Record vollständig.
  - [ ] Koordinatenmapping.
  - [ ] Auswahl und Resize.
  - [ ] Keine Clientautorität.
- [ ] [Kapitel 20: Godot-Netzwerkadapter](../chapters/20-godot-networking.md) — DoD und Completion Record vollständig.
  - [ ] HTTP/WS-Polling.
  - [ ] Serialization/Backoff.
  - [ ] Snapshot/Retry synchronisieren.
- [ ] [Kapitel 21: Lobby-Oberfläche](../chapters/21-lobby-ui.md) — DoD und Completion Record vollständig.
  - [ ] Browser/Create/Join-UI.
  - [ ] Private Einladung.
  - [ ] Fehler und Pending-Zustände.
- [ ] [Kapitel 22: Multiplayer-Oberfläche](../chapters/22-multiplayer-ui.md) — DoD und Completion Record vollständig.
  - [ ] Bestätigte Brettanzeige.
  - [ ] Präsentationsuhren.
  - [ ] Reconnect/Ergebnis-UI.

## Phase VI · M6 — Sicherer Betrieb und Beobachtbarkeit

- [ ] [Kapitel 23: Integration und Fault Injection](../chapters/23-integration-testing.md) — DoD und Completion Record vollständig.
  - [ ] Echte Integration.
  - [ ] Fault-Injection-Matrix.
  - [ ] Zwei-Client-E2E.
- [ ] [Kapitel 24: Security und Missbrauchsschutz](../chapters/24-security.md) — DoD und Completion Record vollständig.
  - [ ] Threat Model.
  - [ ] Limits/Origin/Secrets.
  - [ ] Authorization-Angriffe.
- [ ] [Kapitel 25: Deployment, TLS und Backup](../chapters/25-deployment.md) — DoD und Completion Record vollständig.
  - [ ] Compose/Caddy/TLS selbst konfigurieren.
  - [ ] Backup/Restore-Drill.
  - [ ] Release und Rollback.
- [ ] [Kapitel 26: Observability und Shutdown](../chapters/26-observability.md) — DoD und Completion Record vollständig.
  - [ ] Strukturierte Logs/Metriken.
  - [ ] Health/Readiness.
  - [ ] Shutdown/Resource Cleanup.

## Phasen VII und VIII · M7 — Accounts, Elo, Zuschauer und Junqi

### Phase VII — Plattformfeatures

- [ ] [Kapitel 27: Accounts und Profile](../chapters/27-accounts.md) — DoD und Completion Record vollständig.
  - [ ] Account bei stabilem Principal.
  - [ ] Argon2id/Recovery-Codes.
  - [ ] Upgrade ohne Historyverlust.
- [ ] [Kapitel 28: Elo und Matchmaking](../chapters/28-ratings-matchmaking.md) — DoD und Completion Record vollständig.
  - [ ] Elo-Pools/Einmalwertung.
  - [ ] Matchmaking-Tickets.
  - [ ] Cancel/Match-Race.
- [ ] [Kapitel 29: Zuschauer und Replay](../chapters/29-spectators-replay.md) — DoD und Completion Record vollständig.
  - [ ] Readonly Zuschauer.
  - [ ] History und Replay.
  - [ ] Private Views schützen.
### Phase VIII — Mehrere Spiele

- [ ] [Kapitel 30: Junqi und Mehrspielarchitektur](../chapters/30-junqi.md) — DoD und Completion Record vollständig.
  - [ ] Custom-Graph/Deployment.
  - [ ] Combat und Ergebnisse.
  - [ ] Redaction/Noninterference.

## Fortschrittspflege

Nach jeder Sitzung Kapiteljournal aktualisieren. Nach jedem Kapitel die drei Hauptaufgaben und DoD nur mit nachvollziehbarer Evidenz abhaken. Nach bestandenem Milestone Gate-Zeile und Zähler aktualisieren. Technische Schulden mit Ursache, Auswirkung und vorgesehenem Kapitel dokumentieren; kritische Regel-/Sicherheits-/Durability-Verletzungen sperren den nächsten Meilenstein.
