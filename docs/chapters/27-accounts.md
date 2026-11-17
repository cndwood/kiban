# Kapitel 27 — Registrierte Konten, Profile und Gastübernahme

[Dokumentationsindex](../INDEX.md) · [Projektfortschritt](../checklists/project-progress.md)

## 1. Chapter Overview

Du ergänzt dauerhafte Accounts, ohne die bestehende Gastfunktion zu entfernen. Ein Gast kann seine Identität mit Logincredentials versehen und danach dieselben Partien wiederfinden. Die Plattformidentität bleibt stabil; ein Account ist eine zusätzliche Authentifizierungsmöglichkeit und keine neue Eigentümerschaft über fremde Spiele.

Das Ergebnis umfasst Registrierung beziehungsweise Gastupgrade, Login, Logout, Sessionwiderruf und ein begrenztes öffentliches Profil. Ratings werden erst im nächsten Kapitel hinzugefügt; bisherige Gastpartien werden nicht nachträglich rated.

## 2. Prerequisites

- Kapitel 01–26 abgeschlossen, besonders Sessionpersistenz und Securitytests.
- Datenbanktransaktionen, Eindeutigkeitsconstraints und Password-Hashing-Grundidee kennen; Unterschiede zu Zufallstokenhashes vorab lesen.
- [Identitätsarchitektur](../architecture/authentication.md), [Datenmodell](../database/schema.md) und [Protokollauthentifizierung](../protocol/authentication.md) öffnen.
- Für Tests mehrere echte getrennte Sessions eines Principals sowie einen fremden Account erzeugen können.

## 3. Learning Objectives

- Der Entwickler kann Principal, Account, Session und öffentliches Profil getrennt modellieren.
- Er kann ein Gastupgrade ohne Umschreiben von Spieleigentümerschaft transaktional durchführen.
- Er kann Password-Hashing von Token-Hashing begründet unterscheiden.
- Er kann parallele Registrierung, Sessionrotation und Loginfehler auf Datenlecks und Race Conditions prüfen.

## 4. Concepts and Theory

### Stable Principal

**Definition:** Ein Principal ist die dauerhafte serverseitige Identität, an die Berechtigungen gebunden sind. Ein Account ergänzt Credentials, Sessions sind zeitlich begrenzte Nachweise. **Motivation:** Würde Registrierung eine neue Spieler-ID erzeugen, gingen laufende Sitzrechte verloren. **Einsatz:** Der bestehende Gastprincipal erhält genau einen Account; Spiele und Historie referenzieren ihn unverändert. **Fehler:** Clientseitig angegebene alte Guest-ID als Migrationsbeweis übernehmen. **Geeignet:** Gastupgrade und spätere alternative Loginmethoden. **Vertiefung:** [Domain Model](../architecture/domain-model.md).

### Password Hashing

**Definition:** Ein langsamer, gesalzener Password Hash verteuert Offline-Raten menschlich gewählter Geheimnisse. **Motivation:** Passwörter haben viel weniger Entropie als zufällige Sessiontokens. **Einsatz:** Argon2id mit dokumentiertem Parameterprofil und gespeicherter Versionsinformation; Hasharbeit wird begrenzt außerhalb blockierender Eventlooparbeit ausgeführt. **Fehler:** SHA-256 für Passwörter oder unbegrenzte parallele Hashaufträge. **Geeignet:** Passwortcredentials. **Vertiefung:** OWASP Password Storage und RustCrypto Argon2.

### Credential Binding und Session Rotation

**Definition:** Credential Binding verbindet nachgewiesene Anmeldedaten mit einem Principal; Rotation ersetzt einen Sessionnachweis. **Motivation:** Ein alter gestohlener Gasttoken soll nach Upgrade nicht unbegrenzt das neue Konto kontrollieren. **Einsatz:** Upgrade authentifiziert die aktuelle Session, legt Credentials an und ersetzt den aktuellen Token; andere alte Gastsessions werden widerrufen. **Fehler:** Registrierung nur durch einen Clientbutton sichern oder den neuen Token vor Commit ausgeben. **Geeignet:** Upgrade, Recovery und Passwortwechsel. **Vertiefung:** OWASP Session Management.

### Public Profile Projection

**Definition:** Ein Profil ist eine ausdrücklich erlaubte Teilmenge von Accountdaten. **Motivation:** Eine Datenbankzeile enthält interne Credentials und möglicherweise private Informationen. **Einsatz:** Öffentlich nur Principal-ID, Anzeigename und freigegebene Statistik; niemals Passwort- oder Recoveryhashes. **Fehler:** komplettes Accountobjekt serialisieren und sensitive Felder später per UI verbergen. **Geeignet:** Profil, Leaderboard und Historie. **Vertiefung:** [Security](../architecture/security.md).

## 5. Architectural Context

Authentication bleibt im Server-/Application-Adapter. `Principal` bleibt der Bezugspunkt aller bestehenden Game Seats. Ein persistenter Account verweist eindeutig auf diesen Principal; Sessiontabellen verweisen ebenfalls darauf. Profilabfragen benutzen einen gesonderten Read-DTO.

Beim Upgrade: gültige Gastsession → Credentials validieren und Hash bilden → eindeutigen Account und Sessionrotation in einer Transaktion schreiben → neuen Token einmal ausgeben → offenen Client mit derselben Principal-ID neu authentifizieren. Erfolgt ein Crash nach Commit, bleibt der Account über Login erreichbar. Ein verlorener Tokenresponse darf keine zweite Accountanlage auslösen.

## 6. Design Decisions

**Verbindlich:** Gastupgrade behält den Principal. Kopieren aller Game Seats in einen neuen Account wäre fehleranfällig und würde laufende Actors inkonsistent machen. Die stabile ID vereinfacht Migration, erfordert aber klare Trennung zwischen Gaststatus und Identität. Bei Accountzusammenführung wäre eine neue ADR nötig.

**Verbindlich:** Handle plus Passwort mit Argon2id im ersten Accountumfang. Externes OIDC wäre betrieblich attraktiv, führt aber einen zusätzlichen Anbieter und Verknüpfungsprozess ein. Selbst verwaltete Credentials lehren die Sicherheitsgrenze; ihr Betriebsaufwand muss ernst genommen werden.

**Verbindlich:** Kein automatisches Merge mit einem bereits existierenden Account. Dafür wären Nachweise beider Identitäten und Konfliktregeln nötig. Ein Namenskonflikt beendet ausschließlich das Upgrade, nicht die Gastsession.

**Verbindlich:** Einmalige, kryptografisch zufällige Recoverycodes erlauben Wiederzugang ohne verpflichtenden E-Maildienst. Nur Hashes werden gespeichert; ein Codeverbrauch und Sessionwiderruf sind atomar. Verlorene Codes und verlorenes Passwort sind ohne weiteren verifizierten Nachweis nicht administrativ erratbar. **Optional:** E-Mail-Verifikation oder OIDC später mit eigener ADR.

## 7. Implementation Tasks

- [ ] 1. Ergänze im Domain- und Datenmodell Account, Credential und PublicProfile getrennt vom Principal. Prüfe anhand einer bestehenden Gastpartie, dass deren Seat-Referenz nach Upgrade unverändert bleibt.
- [ ] 2. Definiere Handle-Normalisierung vor der Datenbankanlage. Verwende für Loginhandles ein enges dokumentiertes Zeichenalphabet und Längenbudget; Unicode-Anzeigenamen bleiben davon getrennt und werden als Text dargestellt.
- [ ] 3. Ergänze Eindeutigkeitsconstraints für normalisierten Handle und Account-pro-Principal. Zwei parallele Registrierungen dürfen nicht durch Vorabprüfungen allein abgesichert sein; teste den echten Constraint.
- [ ] 4. Wähle und benchmarke Argon2id-Parameter nach aktueller OWASP-Empfehlung. Dokumentiere Speicher-/Zeitkosten auf dem Zielhost sowie begrenzte Hashparallelität; prüfe Überlastverhalten.
- [ ] 5. Ergänze den Upgrade-Anwendungsfall mit aktueller Gastsession als einzigem Identitätsnachweis. Ein Payload darf keine fremde `principal_id` zum Übernehmen bestimmen; teste manipulierte Eingaben.
- [ ] 6. Schreibe Accountanlage, Gaststatuswechsel und Sessionrotation atomar. Gib den neuen Klartexttoken erst nach Commit aus; ein Failpoint darf weder halbe Credentials noch bestätigten Token zurücklassen.
- [ ] 7. Widerrufe beim Upgrade andere alte Gastsessions des Principals. Lasse aktive Spielsitze bestehen und prüfe, dass der neue Token denselben Sitz erreicht, ein alter Token aber abgelehnt wird.
- [ ] 8. Ergänze Login mit normalisiertem Handle und generischer Fehlerantwort. Unbekannter Handle und falsches Passwort dürfen keinen verlässlichen Accountfinder bilden; prüfe Responseform und begrenzten Aufwand.
- [ ] 9. Ergänze Logout und Übersicht beziehungsweise Widerruf eigener Sessions. Ein fremder Principal darf keine Sessiondaten lesen oder beenden; Session-IDs sind keine Tokens.
- [ ] 10. Implementiere den Recoveryweg über einmalige Zufallscodes. Prüfe einmalige Ausgabe, Hashspeicherung, atomaren Verbrauch, Sessionwiderruf und ungültig machende Neugenerierung; ein Code wird nie per Loginfehler oder Profil ausgegeben.
- [ ] 11. Ergänze öffentliches Profil mit erlaubten Feldern und validiertem Anzeigenamen. Vergleiche DTO mit Datenbankobjekt; Hashes, private Loginmetadaten und Recoverydaten dürfen nicht serialisierbar sein.
- [ ] 12. Ergänze Godot-Flows für Upgrade, Login und Logout. Bei Tokenrotation identischen Principal und aktives Spiel erhalten; bei Logout lokale Secrets löschen und den Verlust aktiver Sitzberechtigung erklären.
- [ ] 13. Ergänze Rate Limits, redigierte Auditereignisse und Reauthentication für Passwortwechsel. Prüfe mindestens einen Brute-Force-, Sessionfixation- und fremden Profiländerungsversuch.
- [ ] 14. Führe Upgrade während einer laufenden Partie mit zwei Sessions aus. Belege unveränderten Seat, verworfene alte Tokens, lesbare alte Historie und weiterhin unrated gespeicherte Gastpartien.

## 8. Public Interfaces and Contracts

| Begriff / Operation | Eingaben und Ergebnis |
|---|---|
| `upgrade_guest` | gültige aktuelle Gastsession, Handle, Passwort, `request_id`; gleicher Principal mit neuem Credential und rotierter Session |
| `login` | normalisierter Handle und Passwort; neuer Sessiontoken oder generischer Loginfehler |
| `logout` | aktuelle Session; serverseitige Revocation und lokales Löschen |
| `PublicProfile` | öffentliche ID, Anzeigename, ausdrücklich freigegebene Statistiken; keine Credentialfelder |
| `change_password` | gültige Session plus erneuter Credentialnachweis; neuer Hash und dokumentierte Sessionrotation |
| Recovery | eigens bestätigter einmaliger Nachweis; atomarer Verbrauch, Credentials-/Sessionerneuerung |

Die konkreten REST-Pfade und Fehlercodes stehen in [REST API](../protocol/rest-api.md). Ein bereits registrierter Principal bekommt durch erneutes Upgrade keinen zweiten Account.

## 9. Edge Cases

- Zwei Tabs upgraden denselben Gast: genau ein Account, der zweite Auftrag erhält vorhandenen Abschluss oder Konflikt.
- Zwei Principals wählen denselben normalisierten Handle: Datenbankconstraint entscheidet; kein halber Account bleibt.
- Commit gelingt, Tokenresponse geht verloren: Login mit gerade gesetztem Passwort ermöglicht Wiederzugang.
- Upgrade während aktiver Partie: Seat bleibt gültig, Socket muss mit neuer Session nachgewiesen werden.
- Recoverycode wird parallel zweimal benutzt: nur ein Verbrauch kann committen.
- Name enthält Markup oder Steuerzeichen: keine Ausführung, kein Log-Spoofing.
- Gestohlene alte Gastsession versucht Passwortwechsel: nach Rotation keine Berechtigung.

## 10. Testing Requirements

| Given | When | Then |
|---|---|---|
| Gast mit abgeschlossener und aktiver Partie | Upgrade erfolgreich | gleicher Principal und unveränderte History-/Seat-Referenzen |
| Zwei Upgradeaufträge desselben Principals | gleichzeitig verarbeitet | genau ein Account |
| Gleicher normalisierter Handle zweier Gäste | parallele Registrierung | ein Erfolg, ein definierter Konflikt, keine Teilmigration |
| Upgradecommmit vor verlorenem Response | Login mit neuem Passwort | Wiederzugang zum gleichen Principal |
| Alter Gasttoken | nach Upgrade benutzt | widerrufen, keine Spielmutation |
| Unbekannter Handle / falsches Passwort | Loginversuch | dieselbe externe Fehlerklasse ohne sensible Details |
| Öffentliche Profilabfrage | Response serialisiert | kein Credential-, Session- oder Recoveryfeld |
| Ein Recoverycode | zwei zeitgleiche Einlösungen | höchstens eine erfolgreiche Credentialerneuerung |

Passworttests prüfen Verhalten und Parameterformat, niemals identische gesalzene Hashwerte. Sicherheitsrelevante Race Tests nutzen echte Datenbankconstraints.

## 11. Definition of Done

- [ ] Gastupgrade bewahrt Principal, Sitze und Historie nachweislich.
- [ ] Argon2id-Parameter, Hashbudget und Updatepfad sind dokumentiert.
- [ ] Login, Logout, Sessionrotation und Revocation sind negativ getestet.
- [ ] Recoverycodes sind vollständig spezifiziert und getestet; Codeverlust und fehlender Nachweis erlauben keine fremde Accountübernahme.
- [ ] Öffentliche Profile enthalten ausschließlich freigegebene Felder.
- [ ] Keine Gastpartie wird nachträglich rated; Account- und UI-Verträge sind aktualisiert.

## 12. Common Pitfalls

Eine Gast-ID im Request beweist keinen Besitz. Nur eine bereits verifizierte Session darf ihr Credential an denselben Principal binden. Ein vorheriger Name-Verfügbarkeitscheck ersetzt keinen eindeutigen Datenbankconstraint.

Hasharbeit auf dem Tokio-Eventloop kann alle Spiele verzögern. Begrenze die Zahl teurer Password-Operationen; ein allgemeiner Threadpool ohne Auftragsbudget verschiebt das Exhaustionproblem lediglich.

## 13. Reflection Questions

1. Warum ist ein Principal stabiler als der Status Gast oder registriert?
2. Welche Race Condition verhindert der Account-pro-Principal-Constraint?
3. Warum benötigt ein Passwort andere Hashverfahren als ein zufälliger Sessiontoken?
4. Welche Rechte sollte ein alter Gasttoken unmittelbar nach Upgrade verlieren?
5. Warum ist automatisches Zusammenführen zweier Accounts ein eigener Anwendungsfall?
6. Wie kann ein verlorener Upgrade-Response behandelt werden, ohne Credentials doppelt anzulegen?

## 14. Further Reading

- [OWASP Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html): Argon2id-Parameter, Salt und Upgradebedarf studieren; am 07.11.2026 abgerufen.
- [RustCrypto Argon2](https://docs.rs/argon2/latest/argon2/): Hashformat und Verifikation der gewählten Version nachlesen, keine eigene Kryptografie erfinden.
- [OWASP Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html): generische Fehler und Reauthentication prüfen.
- [PostgreSQL Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html): Unique- und Foreign-Key-Garantien auf Upgrade anwenden.
- [Sessionmodell](../architecture/authentication.md): Tokenrotation gegen bestehende Gastverträge prüfen.

## 15. Completion Record

- Abschlussdatum: ____________________
- Getroffene Designentscheidungen und Begründungen: ____________________
- Aufgetretene Schwierigkeiten und ihre Ursachen: ____________________
- Durchgeführte Tests und überprüfbare Ergebnisse: ____________________
- Verbleibende technische Schulden mit Priorität: ____________________
- Gelernte Zusammenhänge und nächster Lernschritt: ____________________
