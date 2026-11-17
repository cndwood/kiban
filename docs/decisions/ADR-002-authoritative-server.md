# ADR-002 — Autoritativer Server

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

Status: angenommen für das Curriculum. Datum:07.11.2026. Änderungen benötigen eine neue ADR und eine Migrationsbeschreibung.

## Problem

Ein manipulierter Client darf keine Regeln, Uhren oder Ergebnisse festlegen.

## Betrachtete Alternativen

Peer-to-peer Vertrauen; doppelte Clientvalidierung als Autorität; zentraler Server.

## Entscheidung

Nur der Server akzeptiert Zustandsübergänge und erstellt Results; Clients senden Absichten.

## Vorteile

Einheitliche Regeln und Zeitentscheidung; Reconnect aus einer Wahrheit.

## Nachteile und Konsequenzen

Netzwerklatenz und Serververfügbarkeit beeinflussen das Spielgefühl.

## Eignung für Kiban

Traditionelle rundenbasierte Spiele benötigen keine clientseitige Physiksimulation.

## Erneute Prüfung

Offline-Modus kann lokale Engine nutzen, ohne Online-Autorität zu verändern.

## Nachweis

Die betroffenen Architektur-, Protokoll- und Testdokumente müssen dieselbe Entscheidung abbilden. Ein abweichender Prototyp ist als Experiment zu kennzeichnen und darf die Quality Gates nicht ersetzen.
