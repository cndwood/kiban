# M6 — Sicherer Betrieb und Beobachtbarkeit

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Deliverables

Fault-Injection-Suite, Threat Model, Limits, Compose/Caddy/TLS, Backups, strukturierte Logs/Metriken, Health und Shutdown.

## Enthaltene Kapitel

- [Kapitel 23: Integration und Fault Injection](../chapters/23-integration-testing.md)
- [Kapitel 24: Security und Missbrauchsschutz](../chapters/24-security.md)
- [Kapitel 25: Deployment, TLS und Backup](../chapters/25-deployment.md)
- [Kapitel 26: Observability und Shutdown](../chapters/26-observability.md)

## Abhängigkeiten

M5 bestanden. Vorherige Gates dürfen nicht durch spätere Infrastruktur übersprungen werden. Ein abweichender Versuch ist als Experiment zu dokumentieren.

## Akzeptanztests

Kein Canary-Token in Logs; DB-Port nicht öffentlich; Restore erreicht gemessenes RPO24h/RTO2h-Ziel; Lastprofil100 aktive Partien wird dokumentiert; Graceful Shutdown bestätigt keine uncommitted Moves.

Given alle Voraussetzungen und ein frischer Testlauf, When die aufgeführten Fälle reproduziert werden, Then stimmen fachlicher Zustand, Errorcodes und relevante dauerhafte Daten mit den Spezifikationen überein. Ein bloßer Build-Erfolg genügt nicht. Ergebnisse und Versionen im Milestone-Protokoll festhalten.

## Manueller Nachweis

In isolierter Umgebung Release ausrollen, TLS/WSS prüfen, öffentliche Smoke-Partie spielen, DB-Verbindung stören, Readiness beobachten, Backup restaurieren und Restart-Policy demonstrieren.

Eine zweite Person oder der Lernende mit zeitlichem Abstand muss den dokumentierten Ablauf ohne verborgenes Setup wiederholen können. Erwartete und tatsächliche Ergebnisse getrennt notieren.

## Häufige Blocker

Readiness mit Liveness verwechselt; Secrets im Image; ungeprüftes Backup; Down-Migration als Rollback; fremde Forwarded-Header vertraut.

Bei einem Blocker minimalen Reproducer erstellen, zuständige Architekturgrenze bestimmen und erst dann weiter integrieren. Kein Workaround darf den fachlichen Vertrag still verändern.

## Completion Checklist

- [ ] Alle zugeordneten Kapitel-DoDs nachgewiesen.
- [ ] Deliverables selbst implementiert und dokumentiert.
- [ ] Positive, negative und relevante Grenz-/Race-Tests bestehen.
- [ ] Manueller Nachweis mit konkretem Ergebnis protokolliert.
- [ ] Fehlerbehandlung und sensible Daten geprüft.
- [ ] Technische Schulden bewertet, keine kritische offene Invariante.
- [ ] [Projektfortschritt](../checklists/project-progress.md) aktualisiert.

## Nachweisprotokoll

| Feld | Eintrag |
|---|---|
| Datum und implementierter Commit | |
| Toolchain und Testumgebung | |
| Automatisierte Fälle und Resultate | |
| Manueller Ablauf und Resultat | |
| Abweichungen und technische Schulden | |
| Gate bestanden durch | |
