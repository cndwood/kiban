# M3 — Zwei entfernte Terminalspieler

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Deliverables

HTTP-Server, Gast-Sessions, Lobby, private Einladung, JSON-WebSocket und serialisierte Game Actors. Noch ausdrücklich volatile Speicher-Repositories.

## Enthaltene Kapitel

- [Kapitel 09: Axum und HTTP](../chapters/09-http-server.md)
- [Kapitel 10: Gastidentität und Sessions](../chapters/10-guest-authentication.md)
- [Kapitel 11: Lobby und Einladungen](../chapters/11-lobby-management.md)
- [Kapitel 12: WebSocket-Protokoll](../chapters/12-websocket-protocol.md)
- [Kapitel 13: Game Actors und Synchronisation](../chapters/13-multiplayer-state.md)

## Abhängigkeiten

M2 bestanden. Vorherige Gates dürfen nicht durch spätere Infrastruktur übersprungen werden. Ein abweichender Versuch ist als Experiment zu dokumentieren.

## Akzeptanztests

Zwei Gäste können erstellen/beitreten/ziehen/aufgeben; fremder Principal darf keinen Sitz mutieren; zwei parallele Joins ergeben genau einen Sieger; bounded Queue bleibt begrenzt.

Given alle Voraussetzungen und ein frischer Testlauf, When die aufgeführten Fälle reproduziert werden, Then stimmen fachlicher Zustand, Errorcodes und relevante dauerhafte Daten mit den Spezifikationen überein. Ein bloßer Build-Erfolg genügt nicht. Ergebnisse und Versionen im Milestone-Protokoll festhalten.

## Manueller Nachweis

Server lokal starten, zwei voneinander getrennte Terminalclients verbinden, öffentliches und privates Spiel testen, verbotenen Fremdzug versuchen und kontrollierten Disconnect beobachten.

Eine zweite Person oder der Lernende mit zeitlichem Abstand muss den dokumentierten Ablauf ohne verborgenes Setup wiederholen können. Erwartete und tatsächliche Ergebnisse getrennt notieren.

## Häufige Blocker

Client übernimmt Zugvalidierung; Handler umgeht Actor; unbeschränkte Channels; Token im URL; Verwechslung von Displayname und Principal.

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
