# Integrationstests

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Setup

Ein isoliertes PostgreSQL mit denselben Hauptversionsannahmen wie Deployment, vom Lernenden geschriebenen Migrationen und eigener Testdatenbank. HTTP/WS-Server auf freiem Port starten, Sessions über echte Endpunkte erzeugen. Repository-Fakes reichen für diesen Nachweis nicht, weil Constraints, Isolation und Commitfehler Teil des Vertrags sind.

## Pflichtfälle

1. Given leere DB, When alle Migrationen, Then Schema und Constraints erwartungsgemäß vorhanden; zweimaliger Start führt keine Migration doppelt aus.
2. Given zwei Sessions, When Create und Join, Then genau zwei verschiedene Seats, active Version2 und autorisierte Views.
3. Given akzeptierter ZugV, When Prozess nach Commit vor Broadcast endet, Then History und ReceiptV existieren nach Restart, aktive Partie aborted; Retry bewegt nichts erneut.
4. Given Fehler beim zweiten Schreibschritt, When TX zurückrollt, Then weder neuer Snapshot noch Event/Move/Receipt teilweise vorhanden.
5. Given Commitausgang unbekannt, When Verbindung wieder da, Then Abgleich entscheidet anhand Receipt/Version, keine zweite Aktion.
6. Given privates Spiel, When fremder GET ohne Sitz, Then404 und keine Einladung im Response/Log.
7. Given fertiges rated Ergebnis später, When Rating-Use-Case doppelt konkurriert, Then genau ein Marker und zwei konsistente Ratingänderungen.

## Cleanup und Diagnose

Je Lauf Schema/DB mit sicherem eindeutigem Präfix; Cleanup nur eigene Testressourcen. Worker dürfen keine gemeinsamen Fixture-IDs überschreiben. Fehlerartefakte speichern Query-Korrelation und Versionen, keine Credentials. Bei Timeout zuerst Server-/DB-Zustand erfassen, nicht Sleeps erhöhen. Ausgefallener Broadcast wird über neue Subscription/Snapshot nachgewiesen, nicht über erfolgreiche ursprüngliche Socketzustellung.
