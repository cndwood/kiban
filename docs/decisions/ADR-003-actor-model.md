# ADR-003 — Ein Actor pro Partie

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

Status: angenommen für das Curriculum. Datum:07.11.2026. Änderungen benötigen eine neue ADR und eine Migrationsbeschreibung.

## Problem

Brett, Uhr und Lifecycle müssen in konsistenter Reihenfolge geändert werden.

## Betrachtete Alternativen

Ein globaler Lock; pro Partie Mutex; bounded Actor-Mailbox.

## Entscheidung

Ein exklusiver Game Actor pro aktiver Partie, Tokio Task und bounded Channels.

## Vorteile

Serielle Fachlogik, klare Ownership und nachvollziehbare Race-Tests.

## Nachteile und Konsequenzen

Mailbox-Backpressure, Task-Lifecycle und langsame DB benötigen explizite Regeln.

## Eignung für Kiban

Wenige Commands pro Partie lassen Serialisierung effizient und gut lehrbar bleiben.

## Erneute Prüfung

Gemessene Throughputprobleme oder mehrere Writerprozesse; dann Ownership mit Fencing neu entwerfen.

## Nachweis

Die betroffenen Architektur-, Protokoll- und Testdokumente müssen dieselbe Entscheidung abbilden. Ein abweichender Prototyp ist als Experiment zu kennzeichnen und darf die Quality Gates nicht ersetzen.
