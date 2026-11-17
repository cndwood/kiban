# ADR-006 — Monotone Zeit und Deadline

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

Status: angenommen für das Curriculum. Datum:07.11.2026. Änderungen benötigen eine neue ADR und eine Migrationsbeschreibung.

## Problem

Timer können verspätet aufwachen; Systemzeit kann springen.

## Betrachtete Alternativen

Clientuhr; periodischer Dekrement-Task; monotone Deadline mit Actor-Prüfung.

## Entscheidung

Zeit am Actor-Bearbeitungsbeginn; now>=deadline ergibt Timeout; inkrement nur nichtterminal; Restart aktive Spiele neutral abbrechen.

## Vorteile

Deterministische Tests, O(1)-Berechnung, keine doppelte Timeoutautorität.

## Nachteile und Konsequenzen

Queue-/DB-Latenz beeinflusst Fairness; aktive Spiele überleben Restart nicht.

## Eignung für Kiban

Explizite Grenzen sind zuverlässiger als vermeintliche Zeitrekonstruktion aus Wall Clock.

## Erneute Prüfung

Turnierbetrieb verlangt abgesicherte Arrival-Order oder fortsetzbare Wartung; eigene Fairness-ADR erforderlich.

## Nachweis

Die betroffenen Architektur-, Protokoll- und Testdokumente müssen dieselbe Entscheidung abbilden. Ein abweichender Prototyp ist als Experiment zu kennzeichnen und darf die Quality Gates nicht ersetzen.
