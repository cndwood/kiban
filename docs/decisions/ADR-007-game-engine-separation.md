# ADR-007 — Getrennte Spielengines

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

Status: angenommen für das Curriculum. Datum:07.11.2026. Änderungen benötigen eine neue ADR und eine Migrationsbeschreibung.

## Problem

Junqi hat verborgene Information und andere Phasen als Xiangqi.

## Betrachtete Alternativen

Eine universelle Brettmatrix; kopierte Plattform pro Spiel; getrennte Engines mit gemeinsamen Ports.

## Entscheidung

Eigene State-/Action-/View-Modelle pro Engine; gemeinsame Identität, Lobby, Actor, Protokoll und Repositories.

## Vorteile

Regelinvarianten bleiben typisiert; View-Redaction ist explizit.

## Nachteile und Konsequenzen

Dispatch und Serialization müssen pro Spiel gepflegt werden.

## Eignung für Kiban

Mehrspielvision wird unterstützt ohne unnötige Plugin-Infrastruktur.

## Erneute Prüfung

Drittanbieterengines mit unabhängiger Auslieferung könnten Plugin-Sandbox verlangen.

## Nachweis

Die betroffenen Architektur-, Protokoll- und Testdokumente müssen dieselbe Entscheidung abbilden. Ein abweichender Prototyp ist als Experiment zu kennzeichnen und darf die Quality Gates nicht ersetzen.
