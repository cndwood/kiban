# Roadmap und Abhängigkeiten

[Index](INDEX.md) · [Lernleitfaden](LEARNING_GUIDE.md)

Die Phasen bauen inkrementell aufeinander auf. Nichtlineare Experimente sind möglich, aber die Abnahme folgt der Reihenfolge. Aufwand hängt stark vom Rust-Vorwissen ab; mehrere Monate sind realistischer als ein Wochenendtutorial.

| Phase | Kapitel | Integration und Gate |
|---|---|---|
| I Fundament |01–04 | M1: reines Brettmodell |
| II Xiangqi |05–08 | M2: lokale Partie ohne HTTP |
| III Backend |09–13 | M3: zwei Terminalspieler, Speicher-Repositories |
| IV Zuverlässigkeit |14–17 | M4: Fake Clock, Reconnect, PostgreSQL |
| V Godot |18–22 | M5: vollständiger Gast-MVP |
| VI Produktion |23–26 | M6: TLS, Restore, Security, Monitoring |
| VII Plattformfeatures |27–29 | Teil von M7: Accounts, Elo, Replay |
| VIII Mehrere Spiele |30 | M7: Junqi mit Hidden Information |

## Geordnete Kapitel

1. [Rust-Tooling, Git, Cargo Workspaces und CI](chapters/01-workspace.md)
2. [Anforderungen und Invarianten](chapters/02-requirements.md)
3. [Domain Modeling und Rust Ownership](chapters/03-domain-modeling.md)
4. [Brett und Koordinaten](chapters/04-board-representation.md)
5. [Figuren und Pseudo-Legalität](chapters/05-piece-movement.md)
6. [Schach und legale Züge](chapters/06-legal-moves.md)
7. [Endspiel und Wiederholungen](chapters/07-endgame-rules.md)
8. [Engine-API und lokale Partie](chapters/08-engine-api.md)
9. [Axum und HTTP](chapters/09-http-server.md)
10. [Gastidentität und Sessions](chapters/10-guest-authentication.md)
11. [Lobby und Einladungen](chapters/11-lobby-management.md)
12. [WebSocket-Protokoll](chapters/12-websocket-protocol.md)
13. [Game Actors und Synchronisation](chapters/13-multiplayer-state.md)
14. [Monotone Schachuhren](chapters/14-chess-clock-engine.md)
15. [Deadlines und Zeitpartien](chapters/15-timed-matches.md)
16. [Reconnect und Idempotency](chapters/16-reconnection.md)
17. [PostgreSQL und Recovery](chapters/17-persistence.md)
18. [Godot-Szenen und Services](chapters/18-godot-structure.md)
19. [Brettdarstellung und Eingabe](chapters/19-board-rendering.md)
20. [Godot-Netzwerkadapter](chapters/20-godot-networking.md)
21. [Lobby-Oberfläche](chapters/21-lobby-ui.md)
22. [Multiplayer-Oberfläche](chapters/22-multiplayer-ui.md)
23. [Integration und Fault Injection](chapters/23-integration-testing.md)
24. [Security und Missbrauchsschutz](chapters/24-security.md)
25. [Deployment, TLS und Backup](chapters/25-deployment.md)
26. [Observability und Shutdown](chapters/26-observability.md)
27. [Accounts und Profile](chapters/27-accounts.md)
28. [Elo und Matchmaking](chapters/28-ratings-matchmaking.md)
29. [Zuschauer und Replay](chapters/29-spectators-replay.md)
30. [Junqi und Mehrspielarchitektur](chapters/30-junqi.md)

## Meilensteine

- [M1 — Fundament und Rust-Brettmodell](milestones/M1-foundation.md)
- [M2 — Lokal vollständig spielbare Xiangqi-Engine](milestones/M2-xiangqi-engine.md)
- [M3 — Zwei entfernte Terminalspieler](milestones/M3-multiplayer.md)
- [M4 — Zeit, Reconnect und Persistenz](milestones/M4-timed-games.md)
- [M5 — Vollständig spielbarer Godot-Gast-MVP](milestones/M5-godot-mvp.md)
- [M6 — Sicherer Betrieb und Beobachtbarkeit](milestones/M6-production.md)
- [M7 — Accounts, Elo, Zuschauer und Junqi](milestones/M7-expansion.md)

## Übergabeverträge

Engine→Backend: geprüfte Action, Folgezustand, Outcome ohne IO. Backend→Clock: ein monotoner Entscheidungszeitpunkt pro neuem Command. Clock→Persistenz: Kandidat mit finalen Budgets und Deadlinebasis. Persistenz→Client: erst committed View, danach Ack/Event. Godot→Erweiterungen: View Model kennt game_kind/ruleset_id statt globaler Xiangqi-Annahmen.

Kapitel01 beginnt mit einer selbst zu erstellenden Toolchain/CI; tatsächlich laufende GitHub Actions werden erst benötigt, sobald eigener Code existiert. Terminaladapter bleibt auch nach Godot wichtig für Fehlerdiagnose. Datenbankrollen werden lokal17 eingeführt, Produktionsrechte25. Kein Kapitel benötigt Accounts vor27 oder Redis/Kubernetes zu irgendeinem Pflichtzeitpunkt.
