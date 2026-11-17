# ADR-004 — REST und JSON-WebSocket

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

Status: angenommen für das Curriculum. Datum:07.11.2026. Änderungen benötigen eine neue ADR und eine Migrationsbeschreibung.

## Problem

Ressourcenverwaltung und laufende Ereignisse haben verschiedene Lebenszyklen.

## Betrachtete Alternativen

Nur REST Polling; nur WebSocket RPC; binäres Protokoll; REST plus JSON WS.

## Entscheidung

REST für Sessions/Lobby/History, WS für Spielcommands, Snapshots und Events; protocol_version1.

## Vorteile

Debuggbar mit Terminalclients, interoperabel mit Godot und Browser.

## Nachteile und Konsequenzen

Eigene Request-Receipts, Resync und Versionierung nötig; JSON ist größer.

## Eignung für Kiban

Nachvollziehbarkeit und einfache Tests sind für kleine Brettzustände wichtiger als Bytes.

## Erneute Prüfung

Snapshots überschreiten64KiB oder echte Messwerte rechtfertigen binäre Darstellung.

## Nachweis

Die betroffenen Architektur-, Protokoll- und Testdokumente müssen dieselbe Entscheidung abbilden. Ein abweichender Prototyp ist als Experiment zu kennzeichnen und darf die Quality Gates nicht ersetzen.
