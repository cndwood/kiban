# Teststrategie und Evidenz

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Schichten und Lernreihenfolge

Domain-/Engine-Unit-Tests sind schnell und deterministisch, ohne Async oder DB. Property-Based-Tests prüfen Invarianten über viele gültige Zustände, ersetzen aber keine erklärbaren Regelbeispiele. Application-Tests setzen Fake Clock, Speicher-Repository und kontrollierbare Channels ein. Integrationstests prüfen reale PostgreSQL-Transaktionen und HTTP/WS. End-to-End verbindet zwei echte Clients mit Backend; UI-Checks bleiben wenige kritische Flows.

Eine Testpyramide beschreibt Kosten, nicht pauschale Testquoten. Coverage-Zahlen allein belegen weder korrekte Schachregeln noch sichere Redaction. Entscheidend ist Traceability: R-Kennung→Kapitel→Fixture→erwarteter Nachweis. Jeder behobene fachliche Bug bekommt einen Regressionstest.

## Determinismus

Kein echtes Warten für Uhren. RNG-Seeds und Actor-Barriers steuern reproduzierbare Interleavings. Testdatenbank je Lauf/Worker isolieren; keine gemeinsam mutierte Developer-DB. HTTP-Testport0 oder kontrollierte eindeutige Ports verwenden. Logs mit Testkennung korrelieren, niemals Tokens ausgeben. Fehlertests injizieren Fehler an benannten Grenzen, nicht durch zufälliges Timing.

## Pflicht-Gates

M2: alle Figuren-/Legalitäts-/Outcome-Fälle und Engine ohne Transport. M3: zwei Terminalclients, paralleler Join, forged Principal. M4: Deadline-Races, Duplicate-Retry, Crash nach Commit vor Ack. M5: vollständiger Godot-Gastflow und Snapshot-Replace. M6: Security, Backup-Restore, Health/Shutdown und gemessene Last. M7: genau einmal Elo und Junqi-Noninterference.

Ein Testprotokoll enthält Versionen, Commit, Datenbasis, Schritte, erwartetes/tatsächliches Resultat und Wiederholbarkeit. Dokumentationscheckboxen bleiben offen, bis der Lernende diese Implementierungsnachweise selbst erzeugt. [Unit](unit-tests.md), [Integration](integration-tests.md), [E2E](end-to-end-tests.md), [Concurrency](concurrency-tests.md), [Clock](clock-tests.md), [Security](security-tests.md) konkretisieren.
