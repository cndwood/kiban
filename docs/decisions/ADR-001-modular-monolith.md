# ADR-001 — Modularer Monolith

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

Status: angenommen für das Curriculum. Datum:07.11.2026. Änderungen benötigen eine neue ADR und eine Migrationsbeschreibung.

## Problem

Ein kleines Projekt braucht klare Grenzen ohne verteilte Betriebsprobleme.

## Betrachtete Alternativen

Einzelnes ungegliedertes Crate; Microservices; modularer Monolith.

## Entscheidung

Ein Prozess mit getrennten Domain-, Engine-, Application- und Adapter-Crates.

## Vorteile

Ein Deployment, direkte Aufrufe, lokale Transaktionen und reproduzierbare Tests.

## Nachteile und Konsequenzen

Ein Prozessausfall betrifft alle Spiele; horizontales Scaling ist zunächst ausgeschlossen.

## Eignung für Kiban

Für einen einzelnen lernenden Entwickler überwiegt nachvollziehbare Korrektheit den Nutzen verteilter Dienste.

## Erneute Prüfung

Unabhängige Teams, nachgewiesene Lastengpässe oder zwingende getrennte Auslieferungen.

## Nachweis

Die betroffenen Architektur-, Protokoll- und Testdokumente müssen dieselbe Entscheidung abbilden. Ein abweichender Prototyp ist als Experiment zu kennzeichnen und darf die Quality Gates nicht ersetzen.
