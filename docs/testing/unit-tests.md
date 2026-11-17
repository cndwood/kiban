# Unit-Tests

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Engine und Domain

Given Rot am Zug und ein gültiger Soldier, When ein erlaubter Vorwärtsschritt geprüft wird, Then genau dieser Kandidat und keine Rückwärtsaktion. Given Cannon ohne Screen und gegnerisches Ziel, When Capture validiert, Then illegal_move und unverändertes Brett. Alle [Positionsfixtures](../rules/rule-test-positions.md) sind tabellengetrieben, je erwarteter Ursache nicht nur pauschal „Fehler“.

Given ein Domain-Identifier im ungültigen Wireformat, When validiert, Then kein ungültiger Domain-Wert. Given Finished-State, When eine neue Aktion, Then invalid_state ohne Versionsänderung. Given gleicher Boardinhalt in anderer Speicherreihenfolge, When Repetition-Key, Then gleiche semantische Position. Given Hashkollision, When Vergleich, Then tatsächlicher Boardvergleich schützt Ergebnis.

## Property-Based Testing

Erzeuge gültige Zustände durch Startposition plus validierte Zugfolgen; minimierte Fehlersequenz speichern. Prüfe Materialgrenzen, genau einen General pro Seite, eingehaltene Palast-/Flussinvarianten, keine eigene Generalbedrohung nach legalem Zug und deterministische Ausgabe bei gleicher Eingabe. Bei Junqi prüfe Materialmultiset und Projektionsgleichheit für unbekannte gegnerische Rangvariationen.

## Testdoubles und Grenzen

Fake Clock für Zeit, Repository-Fake für definierte sichere Fehler, deterministischer Zufallsgeber für IDs in Tests. Keine DB-Details in Engine-Tests. Reine Mock-Erwartung „Methode wurde aufgerufen“ ist kein fachliches Ergebnis. Prüfe Zustand und Fehlersemantik. Tests spiegeln den Vertrag, nicht private Funktionsstruktur; Refactoring darf sie nicht unnötig brechen.
