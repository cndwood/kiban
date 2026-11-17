# Migrationsplan

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Lernreihenfolge

1. Kapitel17 erstellt selbst eine erste Migration für Principal, Session und Spielaggregate. Vorher Tabellenbeziehungen und Constraints als Tests spezifizieren.
2. Nächste Migration ergänzt Events, Moves und Receipts mit atomarer Nutzung; kein produktiver Zwischenzustand ohne Receipt-Durability.
3. Öffentliche Listen- und Historienindizes mit EXPLAIN auf synthetischen Daten prüfen, nicht jeden Feldindex prophylaktisch erzeugen.
4. Kapitel27 erweitert Principal um Accounts; bestehende GuestIds bleiben erhalten. Backfill nullable beginnen, validieren, dann benötigte Constraints schärfen.
5. Kapitel28 ergänzt Pools, Ratings und unique Rating Applications. Keine erneute Wertung alter Casual-Spiele.
6. Kapitel30 ergänzt game_kind/ruleset-spezifische Snapshotversionen, ohne Xiangqi-Zeilen umzudeuten.

## Regeln

Jede Migration hat Zweck, Vor-/Nachbedingungen, erwartete Sperrzeit, Datenmenge, Backupbedarf und Rollforward-Strategie. Bereits ausgeführte Dateien werden nicht nachträglich umgeschrieben. Destruktive Down-Migration ist keine sichere Rollbacklösung. Neue Version liest altes und neues Format während einer kontrollierten Übergangsphase; erst danach alte Spalten entfernen.

SQLx Compile-Time-Queries benötigen passende Schema-Basis beziehungsweise überprüfte Offline-Metadaten. CI darf lokale Developer-DB nicht voraussetzen. Dependency-Version und SQLx-CLI müssen zusammenpassen. Die Befehle und Migrationen schreibt der Lernende selbst; dieses Handbuch enthält keine fertigen SQL-Skripte.

Nachweis: leere DB bis aktuelle Version migrieren; Snapshot einer Vorversion aktualisieren; Anwendungstests ausführen; beschädigten Backfill erkennen; Restore einer Vorab-Sicherung in isoliertem DB-Namen durchführen. Alle Versuche dokumentieren, einschließlich realer Laufzeit.
