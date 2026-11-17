# Persistenzgrenze

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

Die Runtime besitzt den aktuellen Spielzustand; die Datenbank ist ab Kapitel17 die dauerhafte Wahrheit über bestätigte Übergänge. Ein Snapshot beschreibt ein vollständiges Aggregat zu einer Version; eine Move History beschreibt Züge für Replay; ein Event Log beschreibt auch Aufgabe, Timeout und Abbruch. Diese Begriffe sind nicht austauschbar.

Gewählt: vollständiger aktueller Snapshot pro Spiel, append-only Events mit Version und separate Move-Daten sowie Receipts. Kein vollständiges Event-Sourcing-Framework: Recovery darf aus Snapshot starten und muss History-Konsistenz prüfen. Das kostet redundante Daten, macht aber Lesefälle und Anfänger-Debugging überschaubar.

Jeder akzeptierte Übergang aktualisiert Snapshot per expected DB version, fügt genau ein Event und Receipt ein und committed atomar. Datenbankfehler vor sicherem Commit führt zu recovery_required; keine bestätigte Änderung und keine fortlaufende Partie mit unbekannter Uhr. Ein unklarer Commit wird über Receipt und Version abgeglichen, niemals mit neuer ID blind wiederholt. Broadcast ist außerhalb der Transaktion; sein Fehlschlag wird über Resync repariert.

Aufbewahrung als Ausgangsvertrag: Sessions nach Ablauf plus7Tage löschen; Spielhistorien und Receipts180Tage, private Einladungen nach Spielbeginn oder24h unbrauchbar; Accountdaten nach dokumentiertem Löschantrag entkoppeln. Löschfristen müssen gemeinsam mit Backups und Datenschutzansprüchen geprüft werden. Innerhalb der Spielaufbewahrung darf Receipt-GC keine Duplicate-Mutation ermöglichen. [Schema](../database/schema.md) und [Recovery](../database/recovery.md) konkretisieren.
