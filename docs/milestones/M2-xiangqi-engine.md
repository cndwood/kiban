# M2 — Lokal vollständig spielbare Xiangqi-Engine

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Deliverables

Alle sieben Figuren, Legalität, Endergebnisse, Casual-Repetition, öffentliche Engine-API und selbst geschriebener Terminaladapter.

## Enthaltene Kapitel

- [Kapitel 05: Figuren und Pseudo-Legalität](../chapters/05-piece-movement.md)
- [Kapitel 06: Schach und legale Züge](../chapters/06-legal-moves.md)
- [Kapitel 07: Endspiel und Wiederholungen](../chapters/07-endgame-rules.md)
- [Kapitel 08: Engine-API und lokale Partie](../chapters/08-engine-api.md)

## Abhängigkeiten

M1 bestanden. Vorherige Gates dürfen nicht durch spätere Infrastruktur übersprungen werden. Ein abweichender Versuch ist als Experiment zu dokumentieren.

## Akzeptanztests

Alle36 Regelbasisfälle plus Spiegelungen bestehen; Matt und Patt verlieren beide für nächste Seite; Repetition folgt exakt Casual-Profil; abgelehnte Züge verändern nichts.

Given alle Voraussetzungen und ein frischer Testlauf, When die aufgeführten Fälle reproduziert werden, Then stimmen fachlicher Zustand, Errorcodes und relevante dauerhafte Daten mit den Spezifikationen überein. Ein bloßer Build-Erfolg genügt nicht. Ergebnisse und Versionen im Milestone-Protokoll festhalten.

## Manueller Nachweis

Zwei lokale Seiten mit Terminaladapter spielen, Horse-Leg und Cannon-Screen demonstrieren, eine Matt-/Patt-Fixture laden und Ergebnis zeigen.

Eine zweite Person oder der Lernende mit zeitlichem Abstand muss den dokumentierten Ablauf ohne verborgenes Setup wiederholen können. Erwartete und tatsächliche Ergebnisse getrennt notieren.

## Häufige Blocker

Attack Detection ruft sich indirekt rekursiv auf; falsche westliche Pattregel; Probezug verändert Eingabebrett; fehlende Repetitionhistory.

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
