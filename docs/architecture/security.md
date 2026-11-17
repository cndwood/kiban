# Sicherheitsmodell

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Schutzgüter und Vertrauensgrenzen

Schutzgüter: Session-Credentials, Sitzrechte, korrekte Ergebnisse, unveröffentlichte Junqi-Ränge, Accounts und Verfügbarkeit. Nicht vertrauenswürdig: Client, JSON, Anzeigenamen, InvitationCode, Client-Zeit, Event-Reihenfolge und Request-Wiederholung. TLS schützt Transport, ersetzt keine Authorization.

Threat Model je Datenfluss: Spoofing durch gestohlene Tokens; Tampering durch fremde GameId; Repudiation durch fehlende Request-Korrelation; Disclosure durch rohe Snapshots; Denial of Service durch Queue-/Payload-Flut; Privilege Escalation durch Zuschauercommands. Jedes Risiko erhält Gegenmaßnahme und Negativtest.

## Konkrete Ausgangspolicies

Sessionerzeugung maximal10/min pro IP, Spielerstellung10/min pro Principal, Join30/min, Commands20/s mit Burst40 pro Principal und zusätzlichem globalem Budget. Dies sind Startwerte für Kapitel24, keine universelle DDoS-Abwehr. Hinter Caddy nur dessen Verbindung als vertrauenswürdigen Proxy behandeln; X-Forwarded-For aus direktem Internet darf Limits nicht umgehen. Einladung128Bit Zufall, nur gehasht gespeichert,24h Gültigkeit, kein Auflisten; fehlgeschlagene Einladungen liefern einheitliches not_found.

UTF-8-Anzeigenamen auf3–32 Zeichen begrenzen, Kontrollzeichen abweisen und bei Rendering escapen. SQLx bindet Parameter; keine SQL-Konkatenation. Log-Felder werden gegen Zeilen-/Control-Injection behandelt. Tokens, Cookieheader, Passwörter und geheime Spielzustände bleiben aus Logs und Metriklabels.

## Review und Grenzen

Hashing schützt DB-Auszug, nicht bereits gestohlene aktive Credentials. Native Binary enthält keine Server-Secrets. Browser-Sessions benötigen Origin/CSRF, CORS-Allowlist allein ist keine Authorization. Fehler offenbaren keine fremden privaten Spiele. Accountpasswörter folgen später Argon2id und Login-Throttling. [Security-Tests](../testing/security-tests.md) sind ein Release-Gate; es wird keine allgemeine Sicherheit allein durch eine Checkliste behauptet.
