# End-to-End-Tests

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Terminal vor Grafik

M3 benötigt zwei kleine vom Lernenden geschriebene Terminalclients oder ein WebSocket-Testwerkzeug plus HTTP-Client. Given zwei Gäste, When A erstellt und B beitritt, Then beide sehen gleiche GameId, Seiten und Version. A zieht, B sieht autoritativen Zustand; Bs falsche Zugreihenfolge bleibt ohne Wirkung. Keine Godot-Abhängigkeit in diesem Nachweis.

## Godot-MVP

Zwei getrennte Clientinstanzen mit getrennten Credentials starten. A erstellt3+2, B findet öffentliche Lobby, beide spielen mindestens vier Halbzüge. Ein illegaler Versuch wird abgewiesen und UI bleibt konsistent. B trennt Netz, Uhr läuft serverseitig weiter; B verbindet, Snapshot ersetzt Zustand. A gibt auf, beide sehen dasselbe Ergebnis und gespeicherte History. Wiederhole privaten Invite-Flow, ohne Spiel in öffentlicher Liste zu sehen.

## Fehlerszenarien

Given Client verliert Ack nach Move, When Reconnect und identischer Retry, Then ein Halbzug, ein Inkrement. Given Serverrestart bei aktiver Partie, Then neutraler Abbruch, keine automatische neue Partie. Given ungültige JSON-Response im kontrollierten Testadapter, Then verständlicher Fehler statt UI-Crash. Given Bildschirmrotation/Resize, Then logischer Schnittpunkt unverändert.

## Erweiterung

Accountupgrade behält eigene Partien. Matchmaking Cancel-Race erzeugt entweder gematchten Seat oder gültige Cancellation, nie beides. Zuschauer kann nie ziehen. Junqi-Zuschauer sieht unbekannte Ränge weder im UI noch im Roh-JSON. Replay nutzt historische Knowledge-Views, keine versehentlich allwissenden Endzustände.

Testprotokoll nennt Betriebssystem, Godot-Version, Backendversion, Latenz-/Fehlerinjektion, Screenshots oder gespeicherte öffentliche JSON-Views und tatsächliches Ergebnis. Keine echten Tokens in Artefakten.
