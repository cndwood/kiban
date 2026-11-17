# Glossar

[Index](INDEX.md)

## Actor Model

Zustand liegt exklusiv bei einem Nachrichtenempfänger; Kiban serialisiert je Partie Commands, ohne je Actor einen OS-Thread zu reservieren.

## Authentication

Prüft Credentials und liefert PrincipalId; sagt allein nichts über Zugrecht oder private History.

## Authorization

Prüft Recht auf konkrete Aktion/Ressource anhand aktueller Rolle und Spielphase.

## Backpressure

Begrenzte Queues machen Überlast sichtbar; busy oder Disconnect ersetzt unbegrenztes Puffern.

## Borrowing

Zeitweises Ausleihen eines Rust-Werts; geteilte unveränderliche oder exklusive veränderliche Referenzen schützen Zugriff.

## Cancellation

Abbruch einer wartenden Operation; nach Enqueue oder beim Commit ist die fachliche Wirkung möglicherweise bereits eingetreten.

## Clock

Injizierbare Quelle monotoner Zeit für laufende Uhren; Wall Clock separat für UTC-Audit.

## Command

Geprüfte Absicht eines Principals, keine Behauptung eines neuen vollständigen Brettzustands.

## Commit

Dauerhafte Bestätigung einer Transaktion; ab Kapitel17 notwendige Voraussetzung jedes erfolgreichen Mutation-Acks.

## Deadline

Monotoner Zeitpunkt, ab dem die aktive Seite verloren hat; now>=deadline ist abgelaufen.

## Domain Model

Fachliche Begriffe und Invarianten unabhängig von HTTP, SQL und Nodes.

## Event

Dauerhaft akzeptierter Übergang; game_event überträgt eine berechtigte vollständige View dieses Versionsstands.

## Fischer Increment

Feste Zeitgutschrift nach akzeptiertem nichtterminalem Zug; kein Bonus bei illegalem/dupliziertem/terminalem Zug.

## Future

Rust-Wert für noch nicht abgeschlossene Berechnung; Runtime pollt ihn, await blockiert nicht automatisch einen OS-Thread.

## Idempotency

Wiederholung derselben Absicht erzeugt denselben fachlichen Effekt; Receipt+Payload-Hash schützt vor doppelten Zügen.

## Invariant

Bedingung, die an jedem gültigen Systemzustand erfüllt sein muss, etwa genau ein General pro Seite.

## Modular Monolith

Ein aus klar begrenzten Modulen bestehendes Deployment; kein Netzwerk zwischen internen Fachmodulen.

## Monotonic Time

Zeitquelle, die nicht durch UTC-Synchronisation zurückspringt; prozessgebundene Instants nicht persistieren.

## Mutex

Exklusiver Zugriff auf einen geschützten Bereich; schützt keine komplette verteilte Transaktion.

## Ownership

Rust-Werte haben eindeutige Verantwortlichkeit für Lebensdauer; Actor besitzt sein Gameaggregat.

## Principal

Stabile authentifizierbare Identität unabhängig von Name, Session und späterem Account.

## Projection

Empfängerabhängiger Ausschnitt des internen Zustands; bei Junqi zentrale Sicherheitsgrenze.

## Pseudo-legal

Figurenbewegung unter Geometrie-/Blockerregeln, noch ohne vollständige eigene General-Sicherheitsprüfung.

## Race Condition

Fehlerhaftes Ergebnis hängt von Interleaving ab, etwa zwei erfolgreiche Joins auf einen freien Sitz.

## Receipt

Persistenter Nachweis einer akzeptierten Mutation mit Requestscope, Payload-Hash und ursprünglichem Ergebnis.

## Recovery

Kontrollierte Wiederherstellung/Abstimmung nach Fehler; aktive Restart-Spiele werden neutral abgebrochen.

## Redaction

Entfernen nicht berechtigter Informationen vor Serialisierung, nicht bloß Verstecken im UI.

## Request ID

Clientseitige opake Absichts-ID; Retry behält dieselbe ID und denselben Payload.

## RwLock

Mehrere gleichzeitige Leser oder ein Writer; bietet keine fachliche Eventordnung.

## Snapshot

Vollständiger Zustand auf einer Version; API-Snapshot ist eine View, interner DB-Snapshot enthält mehr.

## State Machine

Explizite Zustände und erlaubte Übergänge statt unkontrollierter Kombinationen von Bool-Flags.

## State Version

Monotoner Zähler je Gameaggregat, erhöht sich genau einmal je dauerhafter Änderung, nicht je Frame.

## Transaction

Atomare Datenbankeinheit; Game, Event, Move und Receipt werden gemeinsam geschrieben.

## View

Berechtigtes serialisierbares Spielbild eines Empfängers, niemals das ungefilterte interne Aggregat.

## Wall Clock

Kalenderzeit/UTC für Audit und Ablaufdaten; ungeeignet als alleinige aktive Schachuhr.

## WebSocket

Bidirektionaler Kanal nach HTTP-Upgrade; Anwendung muss Auth, Limits, Retry und Resync selbst spezifizieren.
