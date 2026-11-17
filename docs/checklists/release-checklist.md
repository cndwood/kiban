# Release-Checkliste

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Vor dem Release

- [ ] Version, Commit, Toolchain und Dependencies dokumentiert.
- [ ] Alle relevanten Kapitel-/Milestone-Gates bestanden.
- [ ] Migration auf leerer und vorheriger DB getestet; kompatibler Rückweg benannt.
- [ ] Backup und isolierter Restore erfolgreich, RPO/RTO gemessen.
- [ ] Secrets außerhalb Git/Image, Cookie-/Origin-/TLS-Policies geprüft.
- [ ] Payload-/Mailbox-/Rate-Limits und öffentliche Ports kontrolliert.
- [ ] Keine Tokens oder unbekannten Junqi-Ränge in Logs/DTOs.
- [ ] Maintenancefenster und Abbruchpolicy für aktive Partien bekannt.

## Ausrollen und Abnahme

- [ ] Neue Partien während geplanter Wartung gesperrt, vorhandene drainiert.
- [ ] Image eindeutig versioniert, Migration kontrolliert angewandt.
- [ ] Startup-Recovery abgeschlossen, Readiness erfolgreich.
- [ ] Zwei-Client-Create/Join/Move/Reconnect/Resign via HTTPS/WSS bestanden.
- [ ] Private Einladung und History-Rechte geprüft.
- [ ] Logs, Fehlerquote, Commit-/Queue-Latenz und Ressourcen geprüft.

## Nach dem Release

- [ ] Beobachtungsfenster und verantwortlicher Betreiber dokumentiert.
- [ ] Bei Fehler Rollback/Rollforward nach getesteter Schema-Kompatibilität.
- [ ] Kein unkontrollierter Datenbankrestore über neuere Daten.
- [ ] Incident-/Releaseprotokoll enthält tatsächliche Resultate und Abweichungen.

Diese Checkliste ist später vom Lernenden auszufüllen; das Dokumentationsprojekt selbst deployed keine Anwendung und veröffentlicht nichts.
