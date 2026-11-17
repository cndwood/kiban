# M4 — Zeit, Reconnect und Persistenz

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Deliverables

Injectable Clock, Deadlines, Inkrement, Snapshot/Retry, PostgreSQL, dauerhafte Sessions und History.

## Enthaltene Kapitel

- [Kapitel 14: Monotone Schachuhren](../chapters/14-chess-clock-engine.md)
- [Kapitel 15: Deadlines und Zeitpartien](../chapters/15-timed-matches.md)
- [Kapitel 16: Reconnect und Idempotency](../chapters/16-reconnection.md)
- [Kapitel 17: PostgreSQL und Recovery](../chapters/17-persistence.md)

## Abhängigkeiten

M3 bestanden. Vorherige Gates dürfen nicht durch spätere Infrastruktur übersprungen werden. Ein abweichender Versuch ist als Experiment zu dokumentieren.

## Akzeptanztests

Grenzen999/1000/1001ms, verlorener Ack ohne doppelten Zug/Inkrement, Commitfehler ohne Teilzustand, Neustartabbruch mit erhaltener bestätigter History.

Given alle Voraussetzungen und ein frischer Testlauf, When die aufgeführten Fälle reproduziert werden, Then stimmen fachlicher Zustand, Errorcodes und relevante dauerhafte Daten mit den Spezifikationen überein. Ein bloßer Build-Erfolg genügt nicht. Ergebnisse und Versionen im Milestone-Protokoll festhalten.

## Manueller Nachweis

Zwei Terminalclients spielen3+2; Verbindung nach Commit vor Ack unterbrechen, identisch erneut senden, Uhr weiterlaufen lassen. Server hart stoppen, neu starten, aborted(server_restart) und bestätigte History lesen.

Eine zweite Person oder der Lernende mit zeitlichem Abstand muss den dokumentierten Ablauf ohne verborgenes Setup wiederholen können. Erwartete und tatsächliche Ergebnisse getrennt notieren.

## Häufige Blocker

Echte Sleeps in Uhrtests; Ack vor Commit; Raw-Instant persistiert; neue ID statt Retry; unklaren Commit als sicheren Rollback behandeln.

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
