# Sicherheitstests

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Identität und Zugriff

Given fremde PrincipalId im Payload, When Token einer anderen Session, Then Tokenidentität gewinnt und fremder Sitz ist unzugänglich. Manipuliertes/abgelaufenes/revoked Token wird abgelehnt; Logout sperrt bestehende Socketcommands. Private GameIds und ungültige Einladungen liefern einheitliche404. Ein Zuschauer darf weder make_move noch resign oder Deployment absenden.

## Eingabe und Ressourcen

16KiB-Grenze exakt darunter/darauf/darüber testen, tief geschachteltes JSON begrenzen, fremde Felder und falsche Zahltypen abweisen. Oversized Snapshot ist Server-Contractfehler. Rate Limits an Grenze und nach Ablauf prüfen; gefälschte Forwarded-Header dürfen sie nicht umgehen. Langsame Frames, viele Verbindungen, volle Mailbox und abgebrochene Handshakes dürfen keine unbeschränkten Tasks hinterlassen.

## Browser und Geheimnisse

Fremde Origin mit gültigem Cookie scheitert bei WS und REST-Mutation. Cookie ohne CSRF scheitert. Cookieflags werden im tatsächlichen HTTPS-Deployment geprüft. Native Token bleibt aus URLs. Durchsuche Testlogs und Error-JSON nach Canary-Token, Passwort und unbekanntem Junqi-Rang; Treffer ist Gate-Fehler. SQL-Injection-Eingaben bleiben Daten, Anzeigenamen erzeugen keine Rich-Text-/Log-Injection.

## Hidden Information als Noninterference

Erzeuge zwei interne Junqi-Zustände, die sich nur in unbekannten Gegner-Rängen unterscheiden. Bei gleichen öffentlichen Angaben müssen Viewer-Snapshots, Events, Acks und Errors identisch sein. Deployment vor beidseitigem Ready nicht sichtbar. Combat offenbart nur definierte Ränge; Commander-Tod Flagposition. Replay und History unterliegen denselben Rechten, auch nach Spielende. Nicht nur fehlende UI-Labels prüfen: Rohbytes und serialisierte Felder prüfen.

Security-Review dokumentiert verbleibende Risiken, beispielsweise Single-Host-Ausfall und gestohlenes natives Gerät. Neue Sicherheitsmechanismen benötigen testspezifische Nachweise, keine bloße Bibliotheksnennung.
