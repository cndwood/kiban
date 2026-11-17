# M5 — Vollständig spielbarer Godot-Gast-MVP

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Deliverables

Godot-Servicegrenzen, statische Boardfixtures, Netzwerkadapter, Lobby, Match-UI und serverseitig bestätigte Clocks/Results.

## Enthaltene Kapitel

- [Kapitel 18: Godot-Szenen und Services](../chapters/18-godot-structure.md)
- [Kapitel 19: Brettdarstellung und Eingabe](../chapters/19-board-rendering.md)
- [Kapitel 20: Godot-Netzwerkadapter](../chapters/20-godot-networking.md)
- [Kapitel 21: Lobby-Oberfläche](../chapters/21-lobby-ui.md)
- [Kapitel 22: Multiplayer-Oberfläche](../chapters/22-multiplayer-ui.md)

## Abhängigkeiten

M4 bestanden. Vorherige Gates dürfen nicht durch spätere Infrastruktur übersprungen werden. Ein abweichender Versuch ist als Experiment zu dokumentieren.

## Akzeptanztests

Zwei Godot-Instanzen absolvieren Create/Join/Move/Disconnect/Reconnect/Resign; Resize und schwarze Perspektive erhalten korrektes Koordinatenmapping; Snapshot ersetzt Zustand.

Given alle Voraussetzungen und ein frischer Testlauf, When die aufgeführten Fälle reproduziert werden, Then stimmen fachlicher Zustand, Errorcodes und relevante dauerhafte Daten mit den Spezifikationen überein. Ein bloßer Build-Erfolg genügt nicht. Ergebnisse und Versionen im Milestone-Protokoll festhalten.

## Manueller Nachweis

Separate Gastprofile verwenden,3+2 aus Lobby starten, vier Halbzüge spielen, Netz trennen und wieder verbinden, Partie beenden und Historie abrufen. Privaten Invite-Flow wiederholen.

Eine zweite Person oder der Lernende mit zeitlichem Abstand muss den dokumentierten Ablauf ohne verborgenes Setup wiederholen können. Erwartete und tatsächliche Ergebnisse getrennt notieren.

## Häufige Blocker

WebSocketPeer nicht gepollt; doppelte Signalregistrierung; optimistischer UI-Zug bleibt nach Ablehnung; Client erklärt Timeout; Tokens im Projekt gespeichert.

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
