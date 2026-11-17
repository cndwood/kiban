# 03 — Domain Modeling, Ownership, Enums und Errors

[Dokumentationsindex](../INDEX.md) · [Fortschritt](../checklists/project-progress.md)

## 1. Chapter Overview

Du überführst die Invarianten aus Kapitel 02 in ein verständliches, typisiertes Domain Model. Das greifbare Ergebnis deiner Eigenimplementierung sind fachliche Werte und kontrollierte Übergänge mit aussagekräftigen Fehlern. Noch gibt es keine HTTP-DTOs, keine Datenbank und keine Async-Runtime im Kern.

Der Schwerpunkt liegt auf dem begründeten Umgang mit Ownership. Nimm dir Zeit, Compilerdiagnosen mit der tatsächlichen Lebensdauer der Werte zu verbinden, statt überall Clones oder Shared Ownership einzuführen.

## 2. Prerequisites

- [Kapitel 01](01-workspace.md) und [Kapitel 02](02-requirements.md), insbesondere Invarianten und State Machine.
- Rust Book: Variablen, Funktionen, Kontrollfluss und erste Struct-Grundlagen selbst durcharbeiten.
- Workspace-Prüfungen funktionieren; du kannst eigenständig Tests schreiben und Compilerdiagnosen lesen.
- Lies [Domain Model](../architecture/domain-model.md) und [Engine-Separation-ADR](../decisions/ADR-007-game-engine-separation.md).

## 3. Learning Objectives

- Du kannst begründen, welcher Wert eine Ressource besitzt und wann ein Borrow ausreicht.
- Du kannst ungültige Kombinationen durch Typwahl und kontrollierte Konstruktion verhindern.
- Du kannst erwartete fachliche Ablehnungen als `Result` ausdrücken, ohne Panics zu verwenden.
- Du kannst Domain Errors von transportbezogenen Statuscodes unterscheiden.

## 4. Concepts and Theory

### Ownership und Borrowing

Ownership ordnet jedem Wert einen Besitzer zu; beim Move geht diese Zuständigkeit über. Borrowing erlaubt Zugriff ohne Besitzübernahme. Ein unveränderlicher Borrow liest, ein exklusiver veränderlicher Borrow verändert. In Kiban besitzt der Spielzustand das Board; eine Validierung soll ihn zunächst nur lesen. Unüberlegte Clones verdecken Zuständigkeiten und können später zu divergierenden Kopien führen. Ownership ist jetzt sinnvoll, weil der Kern noch synchron und überschaubar ist. Lies im Rust Book Kapitel 4 zuerst komplett.

### Enums und Pattern Matching

Ein Enum beschreibt genau eine Alternative einer geschlossenen Menge. Pattern Matching zwingt zur bewussten Behandlung der Varianten. Kiban kann Spielstatus, Seite und Ergebnis als fachliche Alternativen darstellen, statt widersprüchliche Booleans zu kombinieren. Ein häufiger Fehler sind große „sonst“-Arme, die neue Varianten unbemerkt akzeptieren. Nutze vollständige Fallunterscheidung an fachlichen Übergängen; siehe Rust Book Kapitel 6.

### Option und Result

`Option` beschreibt vorhandene oder fehlende Werte ohne Fehlerbehauptung; ein leeres Boardfeld ist ein Beispiel. `Result` unterscheidet Erfolg von einer erwartbaren Ablehnung, etwa ungültige Koordinaten. Die Grenze ist semantisch, nicht bequemlichkeitsgetrieben. `unwrap` auf externen Daten verwandelt normale Fehlbedienung in Prozessabbruch. Für Kiban werden Domain Errors explizit weitergegeben; Netzwerkstatuscodes entstehen erst am Adapter. Lies Rust Book Kapitel 9.

### Newtypes und Smart Constructors

Ein Newtype ist ein eigener fachlicher Typ um eine Repräsentation. Ein kontrollierter Konstruktor lässt nur gültige Werte zu. So sind GameId und PrincipalId auch dann nicht austauschbar, wenn beide intern dieselbe primitive Form besitzen. Diese Technik verhindert nicht alle Laufzeitfehler; sie schützt nur die tatsächlich modellierte Grenze. Private Felder und wenige kontrollierte Operationen sind wichtig, sonst wird die Konstruktorprüfung umgangen. Verwende sie für stabile fachliche Unterscheidungen, nicht für jeden temporären Zähler.

### Entity, Value Object und deterministischer Kern

Eine Entity bleibt über ihre Identität erkennbar; ein Value Object wird über seinen Inhalt verglichen. Eine Partie ist Entity, eine Koordinate ein Value Object. Deterministisch bedeutet: Gleicher Input plus gleicher Zustand liefert denselben Output, ohne versteckte Uhr oder Zufall. Das ermöglicht reproduzierbare Regeltests. Die Session erzeugt später Zufall im Adapter, niemals die Zugvalidierung. Siehe [Domain Model](../architecture/domain-model.md) und Rust Book „Traits“ für abgeleitete Vergleichbarkeit.

## 5. Architectural Context

`kiban-domain` enthält spielübergreifende Identitäten und Werte. Xiangqi-spezifische `Board`, `PieceKind` und Regelzustände gehören nach `kiban-xiangqi`. Die Anwendung wird später Teilnehmerrechte und Infrastruktur koordinieren. Der Kern darf keinen `axum::Response`, keinen SQLx-Pool und keine Godot-Koordinate veröffentlichen.

Ein Input wird zuerst syntaktisch gelesen, dann in gültige fachliche Werte überführt. Danach prüft der fachliche Übergang kontextabhängige Invarianten. Nur der erfolgreiche Übergang erzeugt einen neuen gültigen Zustand. Ein Fehler verändert den bisherigen Zustand nicht.

## 6. Design Decisions

**Verbindlich: explizite Domain Errors.** Problem: erwartbare Ablehnungen müssen testbar und adapterunabhängig sein. Alternative: Strings oder Panics; Strings sind schwer stabil auszuwerten, Panics ungeeignet für normale Ablehnungen. Typisierte Gründe vereinfachen Tests, verlangen aber sorgfältige Kategorien. Passe sie an, wenn ein Grund mehrere fachlich unterschiedliche Situationen vermischt.

**Verbindlich: konkrete Xiangqi-Typen vor allgemeinem Engine-Trait.** Eine vorschnelle generische Abstraktion kann Junqi auf ein ungeeignetes Board zwingen. Konkrete Typen sind leichter zu verstehen, erfordern später aber eine bewusste Adaptergrenze. Prüfe gemeinsame Interfaces erst nach einem zweiten tatsächlichen Spiel. Optional kannst du alternative interne Repräsentationen vergleichen, sofern öffentliche Verträge unverändert bleiben.

## 7. Implementation Tasks

- [ ] 1. Liste alle fachlichen Substantive aus Kapitel 02 auf und ordne sie Entity, Value Object oder technischer Ressource zu. Prüfe insbesondere Session versus Player.
- [ ] 2. Entwerfe in `kiban-domain` getrennte Namen für `GameId`, `PrincipalId` und `SessionId`; verhindere ihre unbeabsichtigte Austauschbarkeit und belege dies durch Compilerfeedback.
- [ ] 3. Definiere gültige Werte für `Side` und ihre Gegenbeziehung im Xiangqi-Kern. Teste, dass zweimaliger Seitenwechsel den Ausgangswert liefert.
- [ ] 4. Modelliere `PieceKind` mit genau sieben Varianten. Halte chinesische Bezeichnungen als Dokumentation, nicht als UI-abhängige Identität fest.
- [ ] 5. Entwerfe einen `Piece` als fachlichen Wert aus Seite und Art; prüfe, dass ein leerer Platz nicht als achte Figurenart modelliert wird.
- [ ] 6. Modelliere den Lebenszyklus so, dass ein terminales Resultat nur im terminalen Zustand verfügbar ist. Schreibe eigene Beispiele für zuvor mögliche ungültige Feldkombinationen.
- [ ] 7. Definiere kontextfreie Konstruktorprüfungen für begrenzte Werte; lasse bei ungültigem Input einen fachlichen Fehler entstehen, keinen stillen Ersatzwert.
- [ ] 8. Lege Error-Kategorien für ungültigen Wert, falschen Zustand und unzulässigen Übergang fest. Halte ihre Ausgabe frei von HTTP-Codes und internen Secrets.
- [ ] 9. Schreibe lesende Domain-Operationen mit Borrows; begründe für jede verändernde Operation, wer den Zustand exklusiv besitzt.
- [ ] 10. Erprobe an einer selbst gewählten Lernfunktion Move gegenüber Borrow. Dokumentiere, weshalb der Compiler eine spätere Nutzung gegebenenfalls ablehnt.
- [ ] 11. Teste, dass ein abgelehnter Übergang sämtliche fachlichen Werte unverändert lässt; vergleiche vor und nach dem Versuch.
- [ ] 12. Prüfe alle öffentlichen Felder und Methoden auf Umgehung der Konstruktoren. Reduziere Sichtbarkeit und stelle den fachlich notwendigen Zugriff gezielt bereit.
- [ ] 13. Führe Formatierung, Clippy und alle bisherigen Tests aus. Prüfe den Dependency Graph erneut auf Infrastruktur im Kern.
- [ ] 14. Ergänze deine Typverantwortungen in der Domain-Notiz mit je einem erlaubten und einem verhinderten Zustand; verknüpfe sie mit Tests.

## 8. Public Interfaces and Contracts

| Name | Verantwortung | Input/Output und Ablehnung |
|---|---|---|
| `GameId`, `PrincipalId`, `SessionId` | getrennte Identitätsräume | syntaktisch gültiger Wert oder Validierungsfehler |
| `Side` | Xiangqi-Partei Red/Black | Gegenseite, ohne globale Zustandsänderung |
| `PieceKind` | eine der sieben Figurenarten | keine Leerfeld-Variante |
| `Piece` | Seite plus Figurenart | unveränderlicher fachlicher Wert |
| `GameStatus` | gültiger Lebenszyklus | nur erlaubte Zustandsübergänge |
| `GameResult` | sportlicher terminaler Ausgang und Grund | Sieg oder Remis mit `final_version`; `cancelled`/`aborted` bleiben neutrale Lifecycle-Enden |
| `DomainError` | erwartbare Ablehnung | stabiler fachlicher Grund, kein HTTP-Response |

Die Tabelle ist ein Verantwortungsvertrag, keine fertige Typdefinition. Identitätserzeugung gehört später in eine vertrauenswürdige Anwendungsschicht; ein ID-String beweist keine Berechtigung.

## 9. Edge Cases

- Ein leerer Name oder unvollständiger Identitätswert kommt von außen: Konstruktion scheitert kontrolliert.
- Ein Ergebnis wird für eine wartende Partie verlangt: kein erfundener Sieger als Default.
- Dieselbe ID wird als anderer Identitätstyp benötigt: bewusste fachliche Abbildung statt impliziter Konvertierung.
- Ein Fehler wird während einer Mutation erkannt: ursprünglichen Zustand erhalten, keine halbe Transition.
- Der Compiler verlangt einen Clone: zuerst Ownership-Design prüfen, bevor Shared Ownership ergänzt wird.

## 10. Testing Requirements

- **Given** Red, **When** zweimal die Gegenseite bestimmt wird, **Then** entsteht wieder Red; entsprechend für Black.
- **Given** ein leeres Feld, **When** sein Inhalt abgefragt wird, **Then** ist das Ergebnis Abwesenheit und keine künstliche Figur.
- **Given** ein ungültiger begrenzter Wert, **When** der fachliche Konstruktor läuft, **Then** kommt ein definierter Fehler ohne Panic.
- **Given** ein terminaler Zustand, **When** ein weiterer aktiver Übergang verlangt wird, **Then** bleibt Zustand und Ergebnis identisch.
- **Given** zwei gleiche Value Objects, **When** sie verglichen werden, **Then** zählt Inhalt statt Speicheradresse.
- **Given** ein fehlerhafter Übergang, **When** alle beobachtbaren Felder verglichen werden, **Then** ist keines verändert.
- **Given** Engine-Tests, **When** sie isoliert laufen, **Then** werden Uhr, Netzwerk und Zufallsquelle nicht verwendet.

## 11. Definition of Done

- [ ] Fachliche Identitäten sind getrennt; Value Objects und Entities sind begründet.
- [ ] Sieben Figurenarten und zwei Seiten sind eindeutig modelliert.
- [ ] Konstruktoren und Sichtbarkeit schützen die dokumentierten Invarianten.
- [ ] Erwartbare Fehler erzeugen keine Panics und keine Infrastrukturtypen.
- [ ] Fehlerpfade beweisen Unveränderlichkeit des Zustands.
- [ ] Ownership-Entscheidungen sind an mindestens zwei Beispielen erläutert.
- [ ] Alle Quality Gates und frühere Tests bestehen.

## 12. Common Pitfalls

Ein `String` für jede Identität erlaubt fachlich sinnlose Vertauschungen. Ein Enum für Status hilft wenig, wenn daneben widersprechende Flags weiter existieren. Zu großzügige Getter mit veränderlichen Referenzen können jede Invariante umgehen. `Clone` ist nicht grundsätzlich falsch; problematisch ist eine unklare zweite Zustandsautorität. Leite Vergleichbarkeit nur ab, wenn ihre fachliche Bedeutung stimmt, und kopple Errors nicht voreilig an die spätere Weboberfläche.

## 13. Reflection Questions

1. Warum ist ein leeres Boardfeld eher `Option` als `Result`?
2. Welche Invarianten kann ein Typ schützen und welche benötigen den aktuellen Spielzustand?
3. Wann ist ein Clone fachlich unproblematisch, wann erzeugt er eine zweite Autorität?
4. Weshalb sollten Domain Errors keine HTTP-Statuscodes besitzen?
5. Welche Konsequenz hat ein öffentlich veränderbares Feld für einen Smart Constructor?
6. Warum wird ein universelles Game-Engine-Trait vor Junqi bewusst vertagt?

## 14. Further Reading

- [Rust Book: Ownership](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html): Move, Referenzen und Borrowing vollständig durcharbeiten.
- [Rust Book: Enums](https://doc.rust-lang.org/book/ch06-00-enums.html): `Option` und vollständiges Matching für Domain-Zustände.
- [Rust Book: Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html): erwartbare Fehler und Panic unterscheiden.
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/): „Naming“ und „Predictability“ für verständliche öffentliche Verträge.
- [Domain Model](../architecture/domain-model.md): plattformweite Begriffe gegen Xiangqi-spezifische Typen prüfen.

## 15. Completion Record

- Abschlussdatum: _vom Entwickler einzutragen_
- Getroffene Designentscheidungen und Begründung: _nach Bearbeitung eintragen_
- Aufgetretene Schwierigkeiten: _nach Bearbeitung eintragen_
- Durchgeführte Tests und Belege: _nach Bearbeitung eintragen_
- Verbleibende technische Schulden: _nach Bearbeitung eintragen_
- Gewonnene Erkenntnisse: _nach Bearbeitung eintragen_
