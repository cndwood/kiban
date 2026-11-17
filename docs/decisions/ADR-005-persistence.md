# ADR-005 — PostgreSQL und Commit vor Ack

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

Status: angenommen für das Curriculum. Datum:07.11.2026. Änderungen benötigen eine neue ADR und eine Migrationsbeschreibung.

## Problem

Bestätigte Züge dürfen bei Crash nicht verschwinden oder doppelt wirken.

## Betrachtete Alternativen

Nur Speicher; asynchrone periodische Snapshots; vollständiges Event Sourcing; synchroner Snapshot plus Events.

## Entscheidung

PostgreSQL speichert Snapshot, Event, Move und Receipt atomar; Actor publiziert danach.

## Vorteile

Durability, SQL-History und eindeutige Retry-Semantik ohne verteilte Transaktion.

## Nachteile und Konsequenzen

DB-Latenz liegt auf dem Zugpfad; Ausfall stoppt betroffene Partie.

## Eignung für Kiban

Ein Monolith mit einer DB kann diese Grenze verständlich korrekt umsetzen.

## Erneute Prüfung

Messbarer Commitengpass oder komplexe historische Projektionen rechtfertigen neuen Entwurf.

## Nachweis

Die betroffenen Architektur-, Protokoll- und Testdokumente müssen dieselbe Entscheidung abbilden. Ein abweichender Prototyp ist als Experiment zu kennzeichnen und darf die Quality Gates nicht ersetzen.
