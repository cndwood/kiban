# M1 — Fundament und Rust-Brettmodell

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Deliverables

Typisierte IDs, validiertes90-Punkte-Brett, dokumentierte Zustandsmaschine, selbst geschriebene Workspace-/CI-Struktur.

## Enthaltene Kapitel

- [Kapitel 01: Rust-Tooling, Git, Cargo Workspaces und CI](../chapters/01-workspace.md)
- [Kapitel 02: Anforderungen und Invarianten](../chapters/02-requirements.md)
- [Kapitel 03: Domain Modeling und Rust Ownership](../chapters/03-domain-modeling.md)
- [Kapitel 04: Brett und Koordinaten](../chapters/04-board-representation.md)

## Abhängigkeiten

kein vorheriger Meilenstein. Vorherige Gates dürfen nicht durch spätere Infrastruktur übersprungen werden. Ein abweichender Versuch ist als Experiment zu dokumentieren.

## Akzeptanztests

Ein Koordinatenpaar außerhalb x0..8/y0..9 wird abgelehnt; Startbrett enthält32 Figuren und je einen General. Clone-/Borrowing-Übungen erklären Ownership ohne globale Zustände.

Given alle Voraussetzungen und ein frischer Testlauf, When die aufgeführten Fälle reproduziert werden, Then stimmen fachlicher Zustand, Errorcodes und relevante dauerhafte Daten mit den Spezifikationen überein. Ein bloßer Build-Erfolg genügt nicht. Ergebnisse und Versionen im Milestone-Protokoll festhalten.

## Manueller Nachweis

Frisches eigenes Repository vorbereiten, selbst Toolchain einrichten, Tests starten, Startaufstellung ausgeben und eine ungültige Koordinate absichtlich prüfen.

Eine zweite Person oder der Lernende mit zeitlichem Abstand muss den dokumentierten Ablauf ohne verborgenes Setup wiederholen können. Erwartete und tatsächliche Ergebnisse getrennt notieren.

## Häufige Blocker

Cargo-Abhängigkeiten kreisförmig; Rust-Lifetimes vor Domainmodell überkompliziert; Bildschirm- und Fachkoordinaten verwechselt.

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
