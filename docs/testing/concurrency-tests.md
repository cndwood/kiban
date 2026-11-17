# Concurrency- und Backpressure-Tests

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Gesteuerte Interleavings

Mit Barrier starten zwei Joins auf denselben freien Sitz. Genau einer gewinnt; Verlierer game_full/version_conflict; ein active-Event, ein Actor, kein Principal doppelt. Zwei Commands desselben Spielers mit gleicher expected_version aber verschiedener ID dürfen nicht beide auf derselben Ausgangsposition wirken.

Given zwei identische request_id+Payload, When gleichzeitig enqueued, Then erstes Commit erzeugt Receipt, zweiter liefert dasselbe Ergebnis ohne Inkrement/Version. Given gleiche ID anderer Payload, Then request_id_conflict. Given Actor schließt und Registry bekommt Ersatzgeneration, When alter Cleanup läuft, Then neuer Handle bleibt erhalten.

## Grenzen und Cancellation

Fülle Mailbox bis Limit, weitere Annahme liefert busy innerhalb begrenzter Zeit und Speicher bleibt beschränkt. Halte Zuschauer-Writer an, Then seine Queue wird getrennt, Spieler bleibt fortschrittsfähig. Breche Client nach Enqueue ab, Then Command wird genau einmal verarbeitet und Receipt bleibt abfragbar. Shutdown während Commit darf keinen Erfolg vor Durability versenden.

## Zeitrennen

Move und Timer werden in beiden Enqueue-Reihenfolgen bei Fake now vor/genau/nach Deadline getestet; maßgeblich ist beim tatsächlichen Bearbeitungsbeginn gemessenes now. Ein alter Timer nach Zugwechsel hat keinen Effekt auf neue Generation. Details in [Clocktests](clock-tests.md).

Stresstests ergänzen deterministische Fälle, ersetzen sie nicht. Ergebnisse umfassen Queue-Länge, p95 Bearbeitungszeit, aktive Tasks nach Ende und Registrygröße. Nach vielen kurzen Partien müssen Tasks und Subscriber zum Ausgangsniveau zurückkehren. Keine echten Sleeps als zufälliger Beweis von Race-Freiheit.
