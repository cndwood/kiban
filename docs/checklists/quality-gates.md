# Quality Gates

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Für jedes Kapitel

- [ ] Voraussetzungen erfüllt, neue Konzepte vor Anwendung erläutert.
- [ ] Alle Implementation Tasks mit überprüfbarem Ergebnis bearbeitet.
- [ ] Given/When/Then-Fälle, Grenzen und relevante Fehlerpfade bestehen.
- [ ] Domain-/Adaptergrenzen eingehalten, keine unkontrollierten panics für Benutzereingaben.
- [ ] fmt/clippy/Tests ohne unverstandene Warnungen; Ausnahmen begründet.
- [ ] Schnittstellen-/Regeländerungen in betroffenen Dokumenten synchronisiert.
- [ ] Completion Record enthält echte Ergebnisse statt bloßem Datum.

## Phasengrenzen

- [ ] M1: typisierte Koordinaten und Startposition korrekt.
- [ ] M2: lokale vollständige Xiangqi-Partie ohne IO in Engine.
- [ ] M3: zwei Terminalclients, Authorization und Actor-Serialisierung.
- [ ] M4: Deadline-Races, Retry und Commit-Durability mit realer DB.
- [ ] M5: vollständiger Godot-Gastflow und statische Renderingtests.
- [ ] M6: Threat Model, Restore, TLS und Betriebsnachweise.
- [ ] M7: Elo genau einmal, Matchmaking atomar, Junqi-Views geheim.

## Sperrkriterien

Ungeklärte doppelte Züge, falsches Timeout, Token-/Rangleak, Ack/Broadcast vor bestätigtem vollständigem Commit oder widersprüchliche ruleset_id sperren das Gate. Ein offener kosmetischer UI-Fehler kann mit dokumentierter Priorität weitergeführt werden. Gateentscheidungen unterscheiden fachliche Korrektheit, Betriebsrisiko und Lernschulden; kein roter Test wird durch Entfernen der Assertion „gelöst“.
