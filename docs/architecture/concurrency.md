# Concurrency und Cancellation

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Grundlagen

Concurrency bedeutet überlappenden Fortschritt; Parallelism bedeutet gleichzeitige Ausführung auf mehreren Kernen. Eine Race Condition hängt von Interleaving ab, auch wenn keine unsicheren Speicherzugriffe existieren. Ownership verhindert viele Data Races, aber keinen doppelten Join oder doppelten Rating-Commit. Ein Mutex schützt einen kritischen Abschnitt; ein RwLock erlaubt mehrere Leser, ist aber kein Ersatz für eine fachliche Transaktion.

Hinter einem await kann die Runtime andere Tasks ausführen. Halte keine synchronen Locks über Netzwerk- oder DB-await. Eine kurze Registry-Sperre darf nur Actor-Handles lesen/ersetzen; keine Engine ausführen. bounded mpsc erzeugt Backpressure, oneshot verbindet Command und Antwort. Verlorener Antwortempfänger darf einen schon angenommenen Command nicht rückgängig machen.

## Konkrete Ordnungen

Lobby: konkurrierende Joins auf denselben letzten Sitz ergeben genau einen Gewinner. Aktivierung und Actor-Registry-Eintrag sind atomar aus Sicht der Lobby; DB-Version schützt zusätzlich gegen konkurrierende Writer. Partie: Actor-Reihenfolge ist verbindlich, nicht Client-Zeit. Datenbank: compare-and-swap auf state_version plus Unique Constraints verhindert stille Doppelwrites.

Cancellation vor Enqueue ist sicher ohne Wirkung. Nach Enqueue ist Ausgang unbekannt bis Receipt/Resync; der Client muss dieselbe request_id wiederverwenden. Graceful Shutdown beendet Annahme, drainiert begrenzt und behandelt verbleibende aktive Spiele nach Restart-Policy. Tokio task abort ist kein Transaktionsrollback.

## Nachweis

Mit Barrier starten zwei Join-Tasks gleichzeitig. Prüfe genau einen belegten letzten Sitz, keine identischen Principals und genau eine Aktivierung. Wiederhole Duplicate Requests mit verlorenen Acks. Prüfe, dass volle Queue Speichermenge begrenzt und langsame Zuschauer keine Spieler blockieren. Deadlock-Diagnose benötigt begrenzte Testtimeouts, keine sleeps als Synchronisierung.
