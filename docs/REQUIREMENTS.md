# Anforderungen und Invarianten

[Index](INDEX.md) · [Roadmap](ROADMAP.md)

## Umfang und Priorität

| Kennung | Muss-Verhalten | Nachweis |
|---|---|---|
| R01 | Gast besitzt stabile verifizierbare Identität ohne Account | Kapitel10,17; Session nach Neustart nutzbar |
| R02 | Öffentliches Spiel erstellen und paginiert finden | Kapitel11,21 |
| R03 | Privates Spiel nur mit Einladung betreten, nicht auflisten | Kapitel11,24 |
| R04 | Genau zwei verschiedene Principals belegen Sitze | konkurrierender Join in Kapitel13 |
| R05 | Xiangqi nur durch Server validiert | Kapitel05–08,13 |
| R06 | 1+0,3+2,5+0,10+5 als Millisekundenwerte | Kapitel14–15 |
| R07 | Aufgabe, Matt, Patt und Zeitüberschreitung liefern genau ein Ergebnis | Kapitel07,15 |
| R08 | Disconnect pausiert Uhr nicht; Reconnect synchronisiert | Kapitel16,20 |
| R09 | Bestätigter Zug überlebt Prozessabbruch | Kapitel17 |
| R10 | Abgeschlossene öffentliche Partie abrufbar, private nur Teilnehmer | Kapitel17,29 |
| R11 | Godot bedient vollständigen Gast-MVP | Kapitel18–22 |
| R12 | Accounts/Profile, Elo/Matchmaking, Zuschauer/Replay, Junqi später | Kapitel27–30 |

## Fachliche Invarianten

Eine Partie hat genau einen ruleset_id und eine unveränderliche Zeitkontrolle. Ein Principal kann nicht beide Sitze derselben Partie besetzen. Nur der aktive Spieler darf ziehen; Zuschauer dürfen nie mutieren. Nach terminalem Ergebnis gibt es keine weiteren Züge oder Uhrenwechsel. Fehler verändern den fachlichen Zustand nicht. state_version steigt genau einmal je dauerhafter Aggregatänderung, nie wegen Heartbeats, Snapshots oder Duplicate Requests.

Jede akzeptierte Mutation ab Kapitel17 umfasst Zustand, Ereignis und Idempotency Receipt in einer Transaktion. Veröffentlichung erfolgt danach. Ein clientseitiges Brett oder Zeitdisplay ist niemals Autorität. Geheime Junqi-Ränge verlassen nur durch berechtigte Projektionen den Server.

## Qualitätsziele mit Messplan

Korrektheit vor Durchsatz: alle Regel- und Race-Fixtures müssen bestehen. Ausgangsprofil: ein Serverprozess, 100 gleichzeitig aktive Partien, 200 Spieler und 100 Zuschauer auf dokumentierter Testhardware. Ziel: p95 Command-Bearbeitung unter100ms außerhalb künstlicher DB-Störungen; messe Queue-Zeit und Commit-Zeit getrennt. Das ist ein Testziel, keine zugesicherte Produktionskapazität.

Verfügbarkeit benötigt explizite Wartungsregeln: Prozessrestart bricht aktive Partien neutral ab. RPO für bestätigte Spielzüge innerhalb intaktem PostgreSQL-Durability-Vertrag null; Backup-RPO24h und Restore-RTO2h zunächst als Übungsziele messen. Kein Hochverfügbarkeitsversprechen. Datenschutz: keine Tokens in Logs, minimale Profildaten, dokumentierte Aufbewahrung.

## Ausgeschlossen und Annahmen

Kein Chat, Geldspiel, Bot-Service, Kubernetes oder verteilter Actor-Betrieb. Native Godot-Auslieferung zuerst; Webexport ist eigener Sicherheits-/Kompatibilitätstest. Gäste sind ungewertet. Xiangqi-Casual ist ausdrücklich nicht WXF-turnierkonform. Junqi beginnt ungetaktet mit einer vollständig veröffentlichten eigenen Variante. Shogi und Go sind Erweiterungsziele, keine zusätzlichen Kapitelimplementierungen.
