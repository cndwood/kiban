# Kapitel 24 — Threat Modeling, Rate Limiting und Missbrauchsschutz

[Dokumentationsindex](../INDEX.md) · [Projektfortschritt](../checklists/project-progress.md)

## 1. Chapter Overview

Du überprüfst das bereits funktionierende MVP aus der Perspektive eines absichtlich fehlerhaften Clients. Das Ergebnis ist ein begründetes Threat Model mit überprüfbaren Schutzmaßnahmen an Session-, Transport-, Actor- und Datenbankgrenzen. Funktionale Korrektheit ersetzt keine Sicherheitsprüfung: Ein legaler Zug kann vom falschen Spieler stammen.

Das Kapitel konzentriert sich auf die reale Angriffsfläche des modularen Monolithen. Es fordert weder eine neue Security-Plattform noch zusätzliche verteilte Infrastruktur.

## 2. Prerequisites

- Kapitel 01–23 einschließlich Authentifizierung, Actor-Verarbeitung und Integrationstests abgeschlossen.
- HTTP-Statuscodes, TLS-Grundidee und Unterschiede zwischen Authentication und Authorization erklären können.
- [Sicherheitsarchitektur](../architecture/security.md), [Sessionmodell](../architecture/authentication.md) und [Security-Tests](../testing/security-tests.md) lesen.
- Angriffsversuche ausschließlich gegen die eigene lokale oder ausdrücklich freigegebene Testumgebung ausführen.

## 3. Learning Objectives

- Der Entwickler kann Assets, Angreiferfähigkeiten, Trust Boundaries und Gegenmaßnahmen für Kiban benennen.
- Er kann einen gültig authentifizierten fremden Spieler daran hindern, Spielzüge oder Historiedaten unberechtigt zu verändern.
- Er kann Limits als konkrete Speicher-, Zeit- und Auftragsbudgets spezifizieren und testen.
- Er kann Credentials, personenbezogene Daten und später verdeckte Spielinformationen aus Logs fernhalten.

## 4. Concepts and Theory

### Threat Model und Trust Boundary

**Definition:** Ein Threat Model verknüpft schützenswerte Werte, mögliche Angreifer und überprüfbare Angriffswege. Eine Trust Boundary trennt Komponenten mit unterschiedlichem Vertrauen. **Motivation:** Ohne Modell verteilt sich Schutz auf zufällige Einzelprüfungen. **Einsatz:** Godot, Browser und öffentliche Netze sind nicht vertrauenswürdig; Backend prüft jede Mutation. **Fehler:** Verschlüsselung mit Berechtigung verwechseln. **Geeignet:** Vor öffentlicher Bereitstellung und bei jedem neuen Datenfluss. **Vertiefung:** OWASP Threat Modeling in Abschnitt 14.

### Authentication, Authorization und Least Privilege

**Definition:** Authentication belegt die Sessionidentität; Authorization entscheidet eine konkrete Handlung; Least Privilege beschränkt Rechte auf den nötigen Umfang. **Motivation:** Ein gültiger Gasttoken berechtigt nicht zu allen Partien. **Einsatz:** REST prüft Session, Application prüft Sitz und Phase, Datenbankrolle erhält keine unnötigen Administrationsrechte. **Fehler:** `principal_id` aus einem JSON-Feld als Beweis akzeptieren. **Geeignet:** Jede Ressource und jedes Kommando. **Vertiefung:** OWASP Authorization Cheat Sheet.

### Resource Exhaustion und Backpressure

**Definition:** Ein Angreifer erschöpft endliche Ressourcen; Backpressure begrenzt Zufluss und Warteschlangen. **Motivation:** Gültige JSON-Aufträge können CPU, Speicher und Datenbankpools überlasten. **Einsatz:** Payloadlimit, begrenzte WS-Queue, Session- und Principal-Limits sowie Join-/Erstellungsbudgets. **Fehler:** Nur IP-Limits setzen oder unbegrenzte Tasks pro Nachricht starten. **Geeignet:** Öffentliche Eingänge und jede asynchrone Queue. **Vertiefung:** OWASP API4 und Tokio Channels.

### Secret Handling

**Definition:** Secrets sind Werte, deren Besitz Zugriff ermöglicht; sie erfordern begrenzte Speicherung und gezielte Löschung. **Motivation:** Ein Logexport darf keine Gastidentitäten übernehmbar machen. **Einsatz:** 256-bit zufällige Gasttokens nur gehasht serverseitig; Passwörter später mit geeignetem langsamen Password Hash. **Fehler:** Sessiontoken wie ein Passwort zu hashen oder Token im URL-Query übertragen. **Geeignet:** Sessions, Datenbankzugang und Backups. **Vertiefung:** OWASP Session Management.

## 5. Architectural Context

Am Edge begrenzt der Reverse Proxy Verbindungen und Requestgrößen. Axum authentifiziert und deserialisiert begrenzt. Die Application prüft Sitz, Spielphase und Requestberechtigung; der Actor serialisiert autorisierte Entscheidungen und prüft vor Mutation erneut den aktuellen Zustand. PostgreSQL erzwingt zusätzlich Eindeutigkeit und Referenzen.

Sicherheitsprüfungen sind mehrschichtig, aber haben klare Zuständigkeiten. Der reine Xiangqi-Kern kennt keine Tokens. Der UI-Client darf Fehlermeldungen freundlich darstellen, besitzt jedoch keine Sicherheitsautorität.

## 6. Design Decisions

**Verbindlich:** Opaque Gasttokens statt selbstgebautem JWT-System. Stateful Sessions erleichtern Revocation, verlangen aber einen Sessionlookup. Für einen Monolithen ist das ein günstiger Tausch; erst bei belegtem verteiltem Bedarf neu bewerten.

**Verbindlich:** Limits pro IP und Principal sowie globale Obergrenzen. Nur IP zu begrenzen bestraft gemeinsame Anschlüsse; nur Principals zu begrenzen lässt unbegrenzte Gasterstellung zu. Kombinierte Budgets sind nachvollziehbar, erfordern aber eine dokumentierte Proxyvertrauensgrenze.

**Verbindlich:** WebSocket-Authentifizierung nach dem zentralen Protokoll, mit kurzem unauthentifiziertem Handshakebudget und Origin-Prüfung für Webclients. Browser können nicht beliebige Handshakeheader setzen; Tokens gehören trotzdem weder in URLs noch in Logs. Cookiebetrieb erfordert zusätzlich CSRF-/SameSite-Entscheidungen.

**Optional:** Externe Bot-Erkennung erst nach gemessenem Missbrauch; keine Pflicht zu CAPTCHA oder Drittanbieterdaten.

## 7. Implementation Tasks

- [ ] 1. Zeichne in der Sicherheitsdokumentation alle externen Eingänge und Trust Boundaries. Ordne Sessiontoken, private Partien, Spielergebnisse und Verfügbarkeit als Assets zu; prüfe den Datenfluss gegen Architektur und Protokoll.
- [ ] 2. Erstelle je Eingang mindestens einen Spoofing-, Tampering- und Exhaustion-Fall. Notiere Vorbedingung, Wirkung und Gegenmaßnahme; weise jedem Fall einen konkreten Test aus Kapitel 23 zu.
- [ ] 3. Kontrolliere alle mutierenden REST- und WS-Operationen auf serverseitige Principalbindung. Manipuliere Spieler-ID und Farbe im Payload; der Sitz muss weiterhin ausschließlich aus der Session folgen.
- [ ] 4. Prüfe Ablauf und Revocation der Gastsession. Bereits offene Sockets dürfen eine widerrufene Session nicht dauerhaft weiterverwenden; definiere erneute Prüfung bei Kommandos und Sessionende.
- [ ] 5. Erzwinge die kanonischen Payload-, Verbindungs- und Queuegrenzen an Transport und Application. Prüfe Grenzwert und eine Einheit darüber; belege begrenzten Speicher statt nur einen Fehlertext.
- [ ] 6. Ergänze getrennte Budgets für Sessionerstellung, Spielanlage, Join-Code-Versuche und Spielkommandos. Dokumentiere Zeitfenster, Burst und erwartete Antwort; nutze eine steuerbare Zeitquelle.
- [ ] 7. Definiere vertrauenswürdige Proxyadressen. Ein direkt eingehender gefälschter Forwarded-Header darf IP-Budgets nicht umgehen; prüfe Direktzugriff und erlaubten Proxy getrennt.
- [ ] 8. Begrenze unauthentifizierte WebSocketzeit und Nachrichten vor `authenticate`. Prüfe falsche Reihenfolge, wiederholte Authentifizierung und überlange Frames ohne Actoranlage.
- [ ] 9. Ergänze eine Origin-Allowlist für den vorgesehenen Webclient. Native Clients ohne Origin benötigen die dokumentierte Policy; Origin allein ersetzt niemals den Token.
- [ ] 10. Suche in Logs und Fehlerantworten nach Tokens, Einladungscodes und Datenbankdetails. Führe ein absichtlich markiertes Testsecret durch jeden Fehlerpfad und prüfe dessen Abwesenheit in Artefakten.
- [ ] 11. Prüfe Datenbankaufrufe auf gebundene Parameter und enge Rollenrechte. Ein Spielername mit SQL-Syntax bleibt Text; der Laufzeitbenutzer kann keine Administrationsoperation ausführen.
- [ ] 12. Definiere sichere Speicherung des Tokens für native Clients und Webexport getrennt. Erkläre Verlust, Logout und Shared-Computer-Verhalten; keine selbst erfundene lokale Verschlüsselung als Sicherheitsbeweis.
- [ ] 13. Prüfe Dependencies mit gepflegten Advisory-Werkzeugen und dokumentiere einen Updateprozess. Jede Ausnahme braucht betroffene Version, reale Erreichbarkeit und Wiedervorlagedatum.
- [ ] 14. Führe den negativen Security-Lauf gegen die eigene Testumgebung aus. Dokumentiere verbleibende Risiken, deren Begrenzung und das Release-Gate; korrigiere hochwirksame Lücken vor Kapitel 25.

## 8. Public Interfaces and Contracts

| Sicherheitsgrenze | Vertrag |
|---|---|
| Sessionprüfung | unverifizierter Token liefert keine Principalrechte; abgelaufen oder widerrufen wird abgelehnt |
| Spielkommando | Principal muss aktueller berechtigter Sitz sein; erwartete Version ersetzt keine Autorisierung |
| Rate Limiting | expliziter Fehler und Retryhinweis; keine Mutation des abgelehnten Auftrags |
| Payloadlimit | Ablehnung vor teurer Verarbeitung, gemäß [Protokolllimits](../protocol/overview.md) |
| Clientfehler | stabiler Code, korrelierbare ID, keine Stacktraces oder Secrets |
| Laufzeitrolle | nur benötigte Tabellenoperationen; Migrationen haben getrennte Rechte |

Limits müssen im Protokoll und Betrieb dieselben Werte verwenden. Dieses Kapitel führt keine zweite Liste abweichender Produktionsgrenzen ein.

## 9. Edge Cases

- NAT teilt eine IP zwischen vielen Gästen: Principalbudgets und nachvollziehbare Fehlermeldungen vermindern unbeabsichtigte Sperren.
- Angreifer rotiert Gastidentitäten: Gasterstellung und globale Obergrenzen bleiben nötig.
- Session wird bei aktivem Socket widerrufen: nächstes Kommando muss abgelehnt und Verbindung beendet werden.
- Reverse Proxy wird umgangen: Backendport darf öffentlich nicht direkt erreichbar sein.
- Fehlermeldung enthält serialisierten Request: Credentials würden unabsichtlich geloggt.
- Ein späteres Junqi-Snapshot enthält verdeckte Ränge: Clientverstecken genügt nicht, serverseitige Projektion ist zwingend.

## 10. Testing Requirements

| Given | When | Then |
|---|---|---|
| Gültiger Token eines Zuschauers | `make_move` für fremden Sitz | Authorizationfehler, unveränderte Version |
| Widerrufene Session mit offenem Socket | nächstes Kommando | abgelehnt und Socket nach Policy geschlossen |
| Limit exakt ausgeschöpft | nächster Create-Request | begrenzte Ablehnung, keine neue Partie |
| Nicht vertrauenswürdige Quelladresse | gefälschter Forwarded-Header | keine Umgehung des Quell-IP-Budgets |
| Überlanges WS-Payload | Frame wird empfangen | begrenzter Transportabschluss, keine Actorarbeit |
| Name mit SQL-/Markupzeichen | Profil oder Lobby wird angezeigt | Text bleibt Daten, keine Ausführung |
| Markierter Testtoken | beliebiger Fehlerpfad | Token fehlt in Logs, Metriken und Fehlerantworten |
| Falsche Origin bei Webverbindung | WS-Verbindungsversuch | Ablehnung trotz formal korrektem Upgrade |

Rate-Limit-Tests nutzen virtuelle Zeit. Lasttests messen zugleich begrenzte Queues und gesunde Verarbeitung eines normalen Spielers.

## 11. Definition of Done

- [ ] Threat Model enthält konkrete Angreifer, Assets und Trust Boundaries.
- [ ] Jede mutierende Operation hat negative Authorizationtests.
- [ ] Alle öffentlichen Eingänge besitzen messbare Größen-, Zeit- und Mengenlimits.
- [ ] Logs und Clientfehler bestehen die Secretprüfung.
- [ ] Proxy-, Datenbank- und Sessionrechte entsprechen Least Privilege.
- [ ] Releaseblocker sind behoben; verbleibende Risiken mit nachvollziehbarer Zuständigkeit dokumentiert.

## 12. Common Pitfalls

CORS schützt keinen nativen Client und ersetzt keine Authorization. Ein Einladungscode ist kein Spielercredential. Ein beliebiger `X-Forwarded-For`-Header ist keine vertrauenswürdige Quelladresse.

Ungehashte Token in einer Datenbank verwandeln einen lesenden Datenabfluss direkt in Sessionübernahmen. Ein teurer Password Hash für hochentropische Zufallstokens löst dagegen das falsche Problem und belastet jeden Lookup. Trenne Credentialtypen.

## 13. Reflection Questions

1. Welche Angriffe bleiben trotz vollständig legaler Xiangqi-Züge möglich?
2. Warum reichen weder reine IP- noch reine Principal-Limits aus?
3. Welche Daten dürfen Fehlerdiagnosen enthalten, ohne Sessionübernahme zu ermöglichen?
4. Weshalb ist Originprüfung für Browser nützlich, aber kein Identitätsbeweis?
5. Wie verändert eine zusätzliche Proxyebene das Rate-Limit-Modell?
6. Welche Sicherheitsinvariante wird durch einen langsamen WebSocket-Consumer bedroht?

## 14. Further Reading

- [OWASP Threat Modeling](https://owasp.org/www-community/Threat_Modeling): Assets und Trust Boundaries für den eigenen Datenfluss ableiten.
- [OWASP Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html): Fehlerantworten und Schutz vor automatisierten Versuchen studieren; am 07.11.2026 abgerufen.
- [OWASP Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html): deny-by-default und Prüfung jeder Ressource auf Kiban übertragen.
- [OWASP Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html): Tokenlebenszyklus und Transport vergleichen.
- [OWASP WebSocket Security](https://cheatsheetseries.owasp.org/cheatsheets/WebSocket_Security_Cheat_Sheet.html): Handshake, Nachrichtengrenzen und Logging gezielt prüfen.

## 15. Completion Record

- Abschlussdatum: ____________________
- Getroffene Designentscheidungen und Begründungen: ____________________
- Aufgetretene Schwierigkeiten und ihre Ursachen: ____________________
- Durchgeführte Tests und überprüfbare Ergebnisse: ____________________
- Verbleibende technische Schulden mit Priorität: ____________________
- Gelernte Zusammenhänge und nächster Lernschritt: ____________________
