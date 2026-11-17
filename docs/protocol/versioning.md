# Versionierung und Kompatibilität

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

protocol_version1 beschreibt Envelope und Commandsemantik; /api/v1 beschreibt REST-Vertrag; ruleset_id beschreibt Fachregeln; state_version ist der Zähler eines einzelnen Aggregats; Schema-Migrationversion beschreibt DB-Struktur. Diese Werte dürfen niemals gleichgesetzt werden.

Unbekannte protocol_version wird vor Mutation mit unsupported_version abgelehnt. Eingehende unbekannte Mutationsfelder werden strikt zurückgewiesen. Clients müssen unbekannte ausgehende optionale Anzeigefelder ignorieren können; neue verpflichtende Felder, neue Bedeutung bestehender Felder oder veränderte Commandsemantik benötigen neue Protokollversion. Neue Regeln benötigen neue ruleset_id und dürfen laufende Spiele nicht umdeuten.

Vor Release Golden JSON Fixtures für Create, Join, Snapshot, Move, Ack, Event und alle Errorcodes validieren. Rust- und Godot-Contracttests konsumieren dieselben vollständigen Beispiele. Kein Codegenerator ist für das Curriculum nötig. Breaking Change verlangt dokumentierte Upgrade-Reihenfolge, befristete Altclientbehandlung und klaren Fehlerdialog statt stiller Deserialisierung in Defaultwerte.
