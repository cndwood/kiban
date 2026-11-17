# Deployment und Betrieb

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Gewählte Topologie

Ein Host betreibt Caddy, Server und PostgreSQL über Docker Compose. Nur80/443 sind öffentlich; Datenbank und Backend liegen im privaten Compose-Netz. Caddy übernimmt Zertifikate und WSS-Upgrade. Datenbankdateien und Caddy-Zertifikate benötigen persistente Volumes. Keine fertigen Konfigurationsdateien werden im Handbuch erzeugt.

Ein Image pro Serverrelease, Multi-stage Build als Lernziel, nicht-root Prozess und gepinnte Baselines. Secrets kommen aus außerhalb Git verwalteten Dateien/Host-Mechanismen; Compose-Environment ist keine vollständige Secret-Sicherheitslösung. GH Actions prüft fmt, clippy, Tests und dokumentierte SQLx-Metadaten; Releases werden nachvollziehbar getaggt.

## Ablauf und Rückweg

Vor Release: Backup erstellen und Restore in isolierter Umgebung prüfen. Migrationen expand/contract planen, additive Schemas zuerst. Aktive Partien vor geplanter Wartung auslaufen lassen, neue Partien sperren. Graceful Shutdown beendet Annahme und wartet begrenzt auf Commits; verbleibende aktive Spiele fallen unter server_restart-Abbruch. Erst danach Server wechseln und Readiness prüfen.

Rollback bedeutet kompatibles vorheriges Image plus Schema, keine ungetestete Down-Migration mit Datenverlust. Ist Schema inkompatibel, benötigt Rollforward oder verifizierten Restore mit ausgewiesenem RPO. Ein einzelner Host bleibt Single Point of Failure; für das MVP ist einfache Wiederherstellbarkeit wichtiger als vorgetäuschte HA.

## Nachweise

TLS-Zertifikatskette und Ablauf, HTTP→HTTPS, WS-Durchleitung, DB nicht öffentlich, keine Secrets in Image-History, Restore-Zeit, Neustartverhalten und manueller Match-Smoketest werden im Releaseprotokoll dokumentiert. Lasttests auf eigener Hardware, nicht gegen fremde Dienste.
