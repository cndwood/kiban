# Kapitel 25 — Docker, TLS, Deployment und Backups

[Dokumentationsindex](../INDEX.md) · [Projektfortschritt](../checklists/project-progress.md)

## 1. Chapter Overview

Du überführst das getestete Backend in einen reproduzierbaren Betrieb auf einem einzelnen Host. Das Ergebnis ist ein von dir selbst geschriebenes Deployment mit Reverse Proxy, TLS, PostgreSQL, persistenten Daten und einer erfolgreich geprobten Wiederherstellung. Dieses Handbuch liefert dafür Verträge und Abnahmetests, keine fertigen Docker- oder Proxydateien.

Ein gestarteter Container ist noch kein erfolgreicher Betrieb. Erst erreichbare Healthchecks, ein vollständiger Spielablauf und ein überprüfbar wiederherstellbares Backup erfüllen das Kapitelziel.

## 2. Prerequisites

- Kapitel 01–24 abgeschlossen, Security-Gate bestanden und lokale PostgreSQL-Recovery erprobt.
- Container, Prozesse, Ports, Volumes und Unix-Dateirechte vor Umsetzung verstehen.
- Eigener Testhost, Domain und Berechtigung zur DNS-/TLS-Einrichtung vorhanden; zunächst isolierte Stagingumgebung nutzen.
- [Deploymentarchitektur](../architecture/deployment.md), [Migrationsplan](../database/migrations-plan.md) und [Releasecheckliste](../checklists/release-checklist.md) lesen.

## 3. Learning Objectives

- Der Entwickler kann Image, Container und persistentes Volume unterscheiden und deren Lebensdauer demonstrieren.
- Er kann HTTP und WebSocket über denselben TLS-Endpunkt betreiben.
- Er kann ein Release mit Migration, Smoke Test und definierter Rollbackgrenze durchführen.
- Er kann RPO und RTO messen und einen Datenbankrestore auf einem leeren Testsystem nachweisen.

## 4. Concepts and Theory

### Immutable Artifact

**Definition:** Ein Releaseartefakt wird einmal gebaut und unverändert in einer Umgebung gestartet. **Motivation:** Ein Build auf dem Produktionshost kann andere Dependencies erhalten. **Einsatz:** CI erzeugt ein eindeutig versioniertes Image; Deployment referenziert einen unveränderlichen Digest. **Fehler:** `latest` als einziges Releaseprotokoll verwenden. **Geeignet:** Reproduzierbare Staging-/Production-Übergänge. **Vertiefung:** Docker Build- und Image-Dokumentation.

### Reverse Proxy und TLS Termination

**Definition:** Ein vorgeschalteter Server beendet TLS und leitet HTTP beziehungsweise WebSocket zum internen Backend weiter. **Motivation:** Zertifikatsverwaltung und öffentliche Ports bleiben an einer Stelle. **Einsatz:** Caddy vor Axum; PostgreSQL ist ausschließlich intern erreichbar. **Fehler:** Backend öffentlich zusätzlich exponieren oder Upgrade-/Idle-Timeouts übersehen. **Geeignet:** Einzelhost-MVP und spätere standardisierte Edge-Infrastruktur. **Vertiefung:** Caddys Automatic HTTPS und reverse_proxy.

### RPO und RTO

**Definition:** Recovery Point Objective begrenzt tolerierbaren Datenverlust, Recovery Time Objective die gewünschte Wiederherstellungsdauer. **Motivation:** „Wir haben Backups“ beschreibt weder Aktualität noch Nutzbarkeit. **Einsatz:** Für den privaten Pilot zunächst RPO höchstens 24 Stunden und RTO höchstens 2 Stunden; diese Ziele werden vor öffentlicher Nutzung explizit bestätigt oder verschärft. **Fehler:** Backupintervall mit garantiertem Recoverypunkt verwechseln. **Geeignet:** Jede Datenhaltung mit Nutzerdaten. **Vertiefung:** PostgreSQL Backup and Restore.

### Migration Compatibility

**Definition:** Anwendung und Schema bleiben während eines kontrollierten Versionswechsels kompatibel. **Motivation:** Ein altes Binary kann ein bereits umgebautes Schema nicht automatisch rückgängig machen. **Einsatz:** Kleine additive Änderungen, Datensicherung und bewusst geplante Wartung. **Fehler:** Rollback des Images als universellen Datenbankrollback behandeln. **Geeignet:** Jedes Release mit Migration. **Vertiefung:** [Migrationsplan](../database/migrations-plan.md).

## 5. Architectural Context

Ein Host betreibt Caddy, ein Kiban-Serverimage und PostgreSQL über Docker Compose. Caddy besitzt die öffentlichen Ports; Backend und Datenbank sprechen im internen Netz. Nur Datenbank, Zertifikatsdaten und erforderliche Backupartefakte benötigen persistente Speicherung. Keine Redis-, Kubernetes- oder Messagingabhängigkeit wird eingeführt.

Der erste Produktionsweg ist der native Godot-Client über HTTPS/WSS. Ein Webexport ist optional und benötigt separat geprüfte Hosting-, Origin- und Browseranforderungen. Geheimnisse kommen aus geschützten Betriebsdateien oder Hostmechanismen, niemals aus dem Image oder Repository.

## 6. Design Decisions

**Verbindlich:** Docker Compose auf einem einzelnen Host statt Kubernetes. Das reduziert Betriebsaufwand und macht Prozess-/Volumegrenzen sichtbar; Hostausfall bleibt ein Single Point of Failure. Neu bewerten, wenn gemessene Verfügbarkeitsziele mehrere Hosts rechtfertigen.

**Verbindlich:** Caddy als Reverse Proxy. Traefik wäre ebenfalls geeignet und stärker auf dynamische Serviceerkennung ausgerichtet. Caddy passt zur kleinen festen Topologie; die Entscheidung bei geänderter Infrastruktur neu prüfen.

**Verbindlich:** Wartungsfenster statt vorgetäuschtem Zero Downtime. Aktive Spiele werden nach der dokumentierten Restartpolicy `aborted(server_restart)` und bleiben unrated. Das vereinfacht korrekte Clock-Recovery, ist aber nutzerseitig sichtbar. Vor jedem Release neue Partien sperren und laufende Spiele möglichst auslaufen lassen.

**Optional:** PITR und Offsite-Replikation nach sicher beherrschtem logischem Backup; kein Ersatz für einen echten Restore-Test.

## 7. Implementation Tasks

- [ ] 1. Erstelle eine Deploymentinventarliste mit Domain, Ports, Volumes, Secrets und verantwortlicher Person. Markiere öffentlich und intern; überprüfe sie gegen das Threat Model aus Kapitel 24.
- [ ] 2. Wähle konkrete unterstützte Toolversionen und fixiere das Backendimage als reproduzierbares Releaseartefakt. Dokumentiere Rust-, PostgreSQL- und Godot-Kompatibilität; ungetestete Updates bleiben außerhalb des Releases.
- [ ] 3. Schreibe selbst den mehrstufigen Containerbuild. Trenne Buildwerkzeuge von Laufzeit, verwende unprivilegierten Benutzer und überprüfe, dass weder Quellsecrets noch unnötige Tools im Image liegen.
- [ ] 4. Beschreibe die Compose-Services mit internen Netzen und persistentem Datenbankvolume. Starte zunächst lokal; entferne Container ohne Volumeentfernung und belege erhaltene Historie.
- [ ] 5. Lege getrennte Runtime- und Migrationscredentials außerhalb des Repositorys an. Prüfe Dateirechte, Startfehler bei fehlenden Secrets und die eingeschränkten Rechte der Runtime-Rolle.
- [ ] 6. Richte Caddy für die Testdomain ein. Prüfe Zertifikatskette, HTTPS-Weiterleitung und WSS-Upgrade; der interne Backendport darf von außen nicht erreichbar sein.
- [ ] 7. Übertrage die kanonischen Payload- und Timeoutlimits sinnvoll auf den Proxy. Führe eine längere gesunde WS-Partie und einen übergroßen Request aus; beides muss die gewünschte Policy zeigen.
- [ ] 8. Definiere den Releaseablauf: neue Partien sperren, drainen, sichern, migrieren, starten, Smoke Test. Beschreibe jede Abbruchbedingung und die Stelle, ab der Image-Rollback allein nicht genügt.
- [ ] 9. Schreibe die Betriebsbefehle für Backup und Restore in deinen Runbook-Notizen. Backupziel liegt außerhalb des Datenbankvolumes und möglichst außerhalb desselben Hosts; prüfe Verschlüsselung und Zugriff.
- [ ] 10. Erzeuge ein Backup mit einer bekannten abgeschlossenen Testpartie. Stelle es auf einer leeren isolierten Datenbank wieder her und vergleiche Spieler, Züge, Ergebnis, Receipts und Migrationsstand.
- [ ] 11. Messe Backupalter und Restorezeit. Vergleiche die gemessenen Werte mit dem gewählten RPO/RTO; ein überschrittenes Ziel blockiert die Freigabe bis Plan oder Verfahren angepasst ist.
- [ ] 12. Führe den geplanten Wartungsrestart bei laufender Testpartie aus. Nach Recovery muss die Partie nachvollziehbar abgebrochen sein; weder Uhrfortsetzung noch Ratingupdate darf erfunden werden.
- [ ] 13. Prüfe vom externen Client Gastanlage, private Einladung, 3+2-Partie, Reconnect und Historie über TLS. Bewahre Release-Digest und redigierte Abnahmedaten auf.
- [ ] 14. Übe ein fehlgeschlagenes Release im Staging. Nutze die dokumentierte Rollback- oder Restoregrenze, prüfe erneut alle Smoke Tests und ergänze die Releasecheckliste um reale Erkenntnisse.

## 8. Public Interfaces and Contracts

| Betriebsgrenze | Erwarteter Vertrag |
|---|---|
| Öffentlicher Endpunkt | HTTPS und WSS unter derselben dokumentierten Domain; keine Token im URL |
| Backendstart | validierte Konfiguration; Fehler vor Bereitschaft statt stiller Defaultsecrets |
| Datenbanknetz | nur interne berechtigte Clients, keine öffentliche Datenbankportfreigabe |
| Releaseartefakt | eindeutiger Digest, Schemaanforderung und zugehörige Dokumentversion |
| Backup | Zeitpunkt, Schema-/Serverversion, Prüfergebnis, verschlüsseltes Ziel und Aufbewahrung |
| Restore | leeres Ziel, verifizierte Historie, gemessene Dauer und protokollierte Datenverlustgrenze |

Health-Endpunkte werden in Kapitel 26 vertieft. Bis dahin dienen vorhandener Prozesscheck, begrenzter Datenbankprobe und vollständiger Smoke Test als Übergang; ein bloß offener TCP-Port beweist keine Bereitschaft.

## 9. Edge Cases

- Zertifikat kann wegen falschem DNS nicht erneuert werden: Ablauf überwachen und Testdomain zuerst nutzen.
- Proxy beendet inaktive WebSockets: Heartbeat-/Idle-Policy zwischen Proxy und Backend abstimmen.
- Volume fehlt oder wird falsch eingebunden: Start darf nicht unbemerkt eine leere Produktionsdatenbank als erfolgreiche Migration behandeln.
- Backup liegt auf derselben ausgefallenen Platte: Hostverlust würde beide Kopien vernichten.
- Schemaänderung ist nicht rückwärtskompatibel: alte Imageversion darf nicht blind starten.
- Festplatte wird voll: Writes schlagen fehl, Actor muss die Persistenzfehlerpolicy einhalten.
- Deployment startet zwei Backendwriter: Einzelinstanzannahme erzwingen; kein verteiltes Actor-Routing improvisieren.

## 10. Testing Requirements

| Given | When | Then |
|---|---|---|
| Staging mit bekannter Historie | Container neu erstellt | Daten bleiben im Volume und sind lesbar |
| Öffentliche Domain | HTTP-Aufruf und WS-Spiel | gültiges HTTPS/WSS, korrekter Upgradefluss |
| Externer Rechner | direkter Zugriff auf Datenbank oder Backendport | Zugriff nicht möglich |
| Fehlendes Secret | Backend startet | klarer Startfehler, keine Bereitschaft |
| Backup bekannter Partie | Restore auf leere Datenbank | gleiche Züge, Ergebnis und Versionswerte |
| Aktive Partie | Wartungsrestart | einmaliger `aborted(server_restart)`, kein Rating |
| Inkompatible Migration | Releaseprüfung | Abbruch vor unkontrolliertem Binary-Rollback |
| Absichtlich fehlgeschlagenes Image | Staging-Rollback | alter kompatibler Stand plus bestandener Smoke Test |

Restore wird als reale Datenbankoperation geprüft. Ein erfolgreicher Backup-Prozessstatus allein ist keine Abnahme.

## 11. Definition of Done

- [ ] Einzelhost-Deployment ist aus selbst geschriebenen, versionierten Betriebsunterlagen reproduzierbar.
- [ ] HTTPS und WSS funktionieren mit zwei externen Clients.
- [ ] Secrets und Datenbank sind nicht öffentlich oder im Image enthalten.
- [ ] Backuprestore ist erfolgreich und RPO/RTO sind gemessen.
- [ ] Wartungsrestart, Schemaübergang und Rollbackgrenzen sind geprobt.
- [ ] Releasecheckliste enthält Digest, Smoke Tests und offene betriebliche Einschränkungen.

## 12. Common Pitfalls

Ein Docker-Volume ist keine Sicherung. Ein gelöschter Host oder eine fehlerhafte Datenänderung wirkt auch auf persistente Volumes. Backups brauchen getrennte Speicherung und Restoreübungen.

Ein Containerrestart macht eine In-Memory-Uhr nicht rekonstruierbar. Kiban verspricht in diesem Kurs deshalb einen dokumentierten Abbruch aktiver Partien nach Neustart. Ein späteres Resume-Protokoll wäre eine neue Architekturentscheidung, kein kleiner Deploymentparameter.

## 13. Reflection Questions

1. Warum sind Image-Digest und Migrationsstand gemeinsam für einen Rollback nötig?
2. Welcher Ausfall widerlegt ein Backup auf demselben Datenbankvolume?
3. Woran erkennst du den Unterschied zwischen laufendem Prozess und spielbereitem System?
4. Welche Nutzerauswirkung hat die Restartpolicy während eines Deployments?
5. Warum kann eine Proxykonfiguration korrektes HTTP und fehlerhaftes WebSocket zugleich liefern?
6. Welche gemessene Anforderung würde einen zweiten Host rechtfertigen?

## 14. Further Reading

- [Docker Compose](https://docs.docker.com/compose/): Service-, Netz- und Volumelebensdauer verstehen; am 07.11.2026 abgerufen.
- [Docker Build best practices](https://docs.docker.com/build/building/best-practices/): Buildstufen, minimale Laufzeit und reproduzierbare Artefakte studieren.
- [Caddy Automatic HTTPS](https://caddyserver.com/docs/automatic-https): Voraussetzungen und Zertifikatserneuerung prüfen.
- [Caddy reverse_proxy](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy): WebSocket- und Timeoutverhalten nachlesen.
- [PostgreSQL Backup and Restore](https://www.postgresql.org/docs/current/backup.html): logischen Dump von Dateisicherung und PITR unterscheiden; am 07.11.2026 abgerufen.

## 15. Completion Record

- Abschlussdatum: ____________________
- Getroffene Designentscheidungen und Begründungen: ____________________
- Aufgetretene Schwierigkeiten und ihre Ursachen: ____________________
- Durchgeführte Tests und überprüfbare Ergebnisse: ____________________
- Verbleibende technische Schulden mit Priorität: ____________________
- Gelernte Zusammenhänge und nächster Lernschritt: ____________________
