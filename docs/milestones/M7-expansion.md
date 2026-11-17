# M7 — Accounts, Elo, Zuschauer und Junqi

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Deliverables

Accountupgrade bei stabiler Identität, Profile/Recovery-Codes, einmaliges Elo, Matchmaking, redigierter Replay und eigene Junqi-Engine.

## Enthaltene Kapitel

- [Kapitel 27: Accounts und Profile](../chapters/27-accounts.md)
- [Kapitel 28: Elo und Matchmaking](../chapters/28-ratings-matchmaking.md)
- [Kapitel 29: Zuschauer und Replay](../chapters/29-spectators-replay.md)
- [Kapitel 30: Junqi und Mehrspielarchitektur](../chapters/30-junqi.md)

## Abhängigkeiten

M6 bestanden. Vorherige Gates dürfen nicht durch spätere Infrastruktur übersprungen werden. Ein abweichender Versuch ist als Experiment zu dokumentieren.

## Akzeptanztests

Upgrade verliert keine Seats; Ratingmarker verhindert Doppelupdate; Cancel/Match-Race hat einen Ausgang; Zuschauer bleibt readonly; Junqi-Views enthalten keine unbekannten Ränge.

Given alle Voraussetzungen und ein frischer Testlauf, When die aufgeführten Fälle reproduziert werden, Then stimmen fachlicher Zustand, Errorcodes und relevante dauerhafte Daten mit den Spezifikationen überein. Ein bloßer Build-Erfolg genügt nicht. Ergebnisse und Versionen im Milestone-Protokoll festhalten.

## Manueller Nachweis

Gast mit History upgraden, rated Xiangqi matchen, Ergebnis zweimal verarbeiten und unveränderte zweite Wertung zeigen. Zuschauer/Replay öffnen. Junqi Deployment, Combat, Flag-Reveal und verborgene Gegenränge in zwei Views zeigen.

Eine zweite Person oder der Lernende mit zeitlichem Abstand muss den dokumentierten Ablauf ohne verborgenes Setup wiederholen können. Erwartete und tatsächliche Ergebnisse getrennt notieren.

## Häufige Blocker

Guest-Account-Merge anhand Name; Pools vermischt; Rating außerhalb Transaktion; allwissender Replay; Junqi in Xiangqi-Matrix gepresst.

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
