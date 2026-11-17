# Kapitel 28 — Elo, Ratingtransaktionen und Matchmaking

[Dokumentationsindex](../INDEX.md) · [Projektfortschritt](../checklists/project-progress.md)

## 1. Chapter Overview

Du ergänzt faire, nachvollziehbare Gegnervermittlung und persistente Spielstärkeschätzungen. Das Ergebnis sind getrennte Ratingpools, ein genau einmal angewendeter Ergebnisabschluss und eine Matchmakingqueue, die Cancel und Pairing konsistent behandelt. Ein Rating ist eine statistische Schätzung, kein objektives Maß menschlicher Spielqualität.

Die erste Umsetzung verwendet bewusst einfaches Elo. Gäste, abgebrochene Partien und der experimentelle Junqi-Pilot bleiben unrated. Ein Ratingalgorithmus kann korrekt gerechnet und trotzdem falsch integriert sein; das Kapitel legt deshalb besonderes Gewicht auf Transaktionen und Recovery.

## 2. Prerequisites

- Kapitel 01–27 abgeschlossen; stabile registrierte Principals und transaktionale Ergebnisse existieren.
- Fließkommazahlen, Rundung und grundlegende Wahrscheinlichkeit vorab auffrischen.
- [Datenbanktransaktionen](../database/transactions.md), [Concurrency](../architecture/concurrency.md) und [Requirements](../REQUIREMENTS.md) lesen.
- Matchmaking kann zunächst mit Terminalclients ohne neue Godot-Szenen getestet werden.

## 3. Learning Objectives

- Der Entwickler kann Elo und Glicko-2 hinsichtlich Unsicherheit, Komplexität und Datenbedarf vergleichen.
- Er kann ein Ergebnis mit zwei Ratingänderungen und einem Eindeutigkeitsbeleg atomar abschließen.
- Er kann Pairing und Cancel über einen definierten linearen Entscheidungspunkt ordnen.
- Er kann nach Prozesscrash noch nicht angewandte Results nachholen, ohne doppelte Punkte zu vergeben.

## 4. Concepts and Theory

### Elo Expected Score

**Definition:** Elo schätzt aus zwei Ratings eine erwartete Ergebniszahl; das tatsächliche Ergebnis verändert das Rating proportional zur Abweichung. **Motivation:** Siege gegen deutlich stärkere Gegner sollen mehr Information liefern. **Einsatz:** Anfangsrating 1500, konstanter Faktor K=32, Ergebnis 1 für Sieg, 0,5 für Draw, 0 für Niederlage. Erwartungswert ist 1 geteilt durch 1 plus 10 hoch Gegnerdifferenz/400; Änderung ist K mal tatsächliches minus erwartetes Ergebnis. **Fehler:** Ratings nacheinander ändern und für den zweiten Spieler schon den neuen ersten Wert benutzen. **Geeignet:** Klar erklärbarer erster Ratingumfang. **Vertiefung:** FIDE Rating Regulations als Vergleich, ohne FIDE-Kompatibilität zu behaupten.

### Glicko-2 und Uncertainty

**Definition:** Glicko-2 modelliert neben Rating eine Unsicherheit und Volatilität. **Motivation:** Wenige oder lange zurückliegende Ergebnisse sind weniger belastbar. **Einsatz:** Nur Designvergleich und optionale spätere Erweiterung; kein zweites parallel implementiertes System. **Fehler:** Einzelspielupdates ohne bewusst gewählte Ratingperioden als unverändertes Glicko-2 ausgeben. **Geeignet:** Ausreichende Daten und klarer Bedarf an Unsicherheitsschätzung. **Vertiefung:** Glickmans Originalbeschreibung in Abschnitt 14.

### Exactly-once Effect durch Transaktion

**Definition:** Ein wiederholbar zugestellter Abschluss erzeugt durch Eindeutigkeit und atomare Writes genau einen fachlichen Effekt. **Motivation:** Ein Worker kann nach Commit vor Bestätigung abstürzen. **Einsatz:** Eindeutiger Beleg `(game_id, pool_id)` wird mit beiden Ratingänderungen committed. **Fehler:** Erst Punkte schreiben und danach ein separates „processed“-Flag. **Geeignet:** Ratings, Ergebnisstatistik und ähnliche Folgewirkungen. **Vertiefung:** [Transaktionsgrenzen](../database/transactions.md).

### Matchmaking State Machine

**Definition:** Ein Ticket durchläuft `queued`, `paired` oder `cancelled`; ein zentraler Matchmaker serialisiert konkurrierende Entscheidungen. **Motivation:** Ein Cancel darf nicht gleichzeitig mit zwei vergebenen Spielsitzen erfolgreich sein. **Einsatz:** Ein lokaler Matchmaking-Actor besitzt die Queue, die Datenbank sichert Partien und Tickets. **Fehler:** Clientdisconnect als Beweis erfolgreichen Cancels ansehen. **Geeignet:** Einzelinstanz-Monolith; bei mehreren Writern neue Architektur nötig. **Vertiefung:** [Actor Model](../architecture/actor-model.md).

## 5. Architectural Context

Der Game Actor schreibt das terminale Ergebnis unabhängig von der Ratingberechnung. Ein lokaler Backgroundworker sucht persistierte, ratingfähige Results ohne Ratingbeleg. Er sperrt die beiden Poolzeilen in stabiler Principalreihenfolge und schreibt Änderungen plus Beleg in einer Transaktion. Dieser Worker ist kein externer Messagebroker.

Der Matchmaking-Actor ordnet Joinqueue und Cancel. Pairing reserviert zwei kompatible Tickets und erzeugt genau eine persistente Partie mit zugeordneten Seats. Erst nach Commit bekommen beide Clients die `game_id`. Ein Crash vor Commit lässt keinen bestätigten Match zurück; nach Commit kann der aktuelle Ticketstatus wieder abgefragt werden.

## 6. Design Decisions

**Verbindlich:** Elo 1500/K32 als erster Algorithmus. Glicko-2 liefert Unsicherheit, verlangt aber zusätzliche Parameter, Periodenentscheidungen und tiefere Tests. Elo erfüllt den Lernumfang mit leicht prüfbaren Beispielen; bei realer Population und Inaktivitätsproblemen neu bewerten.

**Verbindlich:** Pools nach `game_kind`, `ruleset_id` und Basiszeitbucket: Basiszeit bis einschließlich 300 Sekunden heißt `blitz`, darüber `rapid`. Increment ändert den Bucket hier nicht; dies ist eine Kiban-Policy, kein Turnierstandard. 1+0, 3+2 und 5+0 teilen einen Pool, 10+5 einen anderen. Unterschiedliche Regeln teilen nie Ratings.

**Verbindlich:** Gäste und nachträglich registrierte Gastpartien bleiben unrated. Ratingfähigkeit wird vor Spielbeginn fixiert, um selektives Nachbewerten zu verhindern. Abort-Resultate werden nie gewertet.

**Verbindlich:** Rating intern mit fester dokumentierter Dezimalpräzision speichern und erst zur Anzeige runden. So bleiben entgegengesetzte Änderungen konsistent; eine spätere Präzisionsänderung benötigt reproduzierbare Migration.

**Optional:** Wartezeitabhängige Erweiterung des Ratingfensters. Starte mit ±100 Punkten und erweitere alle 10 Sekunden um 50 bis maximal ±400; messbare Qualität und Wartezeiten entscheiden über spätere Anpassung.

## 7. Implementation Tasks

- [ ] 1. Definiere die Pool-ID aus Spieltyp, Regelprofil und Basiszeitbucket. Lege Grenztests für 300 und 301 Sekunden an; Increment darf die dokumentierte Zuordnung nicht heimlich verändern.
- [ ] 2. Ergänze Ratingfähigkeit als vor Beginn fixierten Spielparameter. Erlaube rated nur für zwei registrierte Principals im unterstützten Xiangqi-Pool; manipulierter Gastrequest wird abgelehnt oder ausdrücklich unrated erstellt.
- [ ] 3. Entwerfe reine Elo-Berechnung mit zwei alten Ratings und Ergebnis als Eingabe. Dokumentiere erwartete Symmetrie, erlaubte Werte und konstante Laufzeit; teste gleiche Ratings vor Datenbankintegration.
- [ ] 4. Lege Anfangswert 1500, K32 und feste Speicherpräzision fest. Prüfe Sieg gleicher Ratings mit 1516/1484 sowie Draw mit unveränderten 1500/1500; zeige Rundung nur in der UI.
- [ ] 5. Ergänze Poolratingzeilen und eindeutigen Ergebnisbeleg `(game_id, pool_id)`. Datenbankconstraints müssen doppelte Belege verhindern; teste denselben Abschluss parallel.
- [ ] 6. Implementiere den Ergebnisworker als lokal überwachte Task. Er sucht abgeschlossene ratingfähige Partien ohne Beleg; ein Restart muss aus der Datenbank fortsetzen können, ohne flüchtige Queue als Wahrheit zu brauchen.
- [ ] 7. Sperre bei einer Ratingtransaktion beide Principalzeilen in stabiler Reihenfolge. Berechne aus demselben alten Wertepaar und schreibe beide Änderungen plus Beleg atomar; ein Fehler darf keine halbe Wertung hinterlassen.
- [ ] 8. Prüfe Abort, unrated und Gastupgrade explizit im Worker. Bestehende Gastpartien bleiben unabhängig vom heutigen Accountstatus unrated; Testdaten müssen diese zeitliche Unterscheidung belegen.
- [ ] 9. Ergänze Matchmakingtickets mit Principal, Pool, genauer Zeitkontrolle, Eintrittszeit und Status. Pro Principal ist höchstens ein aktives Ticket erlaubt; zwei parallele Enqueues dürfen keine Duplikate erzeugen.
- [ ] 10. Implementiere die lokale Matchmaker-Serialisierung. Wähle kompatible Tickets deterministisch nach Wartezeit und stabilem Tiebreak; paare niemals denselben Principal mit sich selbst.
- [ ] 11. Erzeuge Pairing, Seats und Ticketstatus in einer Transaktion. Antworte erst nach Commit mit `game_id`; ein verlorener Response wird durch Ticketstatusabfrage aufgeklärt.
- [ ] 12. Implementiere Cancel als geordnetes Kommando. Vor Pairing wird Ticket cancelled; nach committed Pairing lautet die Antwort bereits paired mit derselben Partie, nicht erfolgreicher Cancel.
- [ ] 13. Ergänze Restartcleanup für ungepaarte Tickets und Abfrage persistenter Pairings. Informiere Clients über verfallene Queueplätze; nach Reconnect ist keine automatische zweite Anmeldung ohne Prüfung erlaubt.
- [ ] 14. Ergänze Profilrating und paginiertes Leaderboard aus den Poolzeilen. Zeige Pool und vorläufigen Status klar; vermeide einen globalen Vergleich über verschiedene Spiele.
- [ ] 15. Ergänze Godot-UI für Queue, Cancel, gefundenes Match und Ratingabschluss. Teste Ergebnis vor Ratingworkerabschluss; ein kurzzeitig ausstehendes Rating ist kein fehlendes Spielresultat.
- [ ] 16. Führe Crash-, Duplicate- und Cancel-Race-Tests aus. Weise je Partie genau einen Ratingbeleg und je Paarung genau eine `game_id` nach; dokumentiere ausgeschlossene Anti-Cheating-Funktionen.

## 8. Public Interfaces and Contracts

| Grenze | Vertrag |
|---|---|
| `calculate_rating_change` | zwei alte Poolratings, gültiges Ergebnis; zwei entgegengesetzte Änderungen ohne IO |
| Ratingbeleg | eindeutiges `(game_id, pool_id)`, alte/neue Werte, Algorithmusversion und Abschlusszeit |
| Queueanmeldung | authentifizierter registrierter Principal, Pool und genaue Zeitkontrolle; eindeutiges aktives Ticket |
| Cancel | Ticket-ID des eigenen Principals; `cancelled` oder bereits `paired` mit `game_id` |
| Matchstatus | persistenter Ticketabschluss, nach Reconnect abfragbar |
| Leaderboard | Pool, Cursor und begrenzte Seitengröße; öffentliche Profilprojektion |

Ratings dürfen zeitlich nach dem Spielergebnis erscheinen. Der Client fragt den Belegstatus ab und rechnet niemals selbst die verbindlichen Punkte. Konkrete HTTP-Verträge stehen in [REST API](../protocol/rest-api.md).

## 9. Edge Cases

- Worker stirbt nach Ratingcommit: erneute Suche findet den vorhandenen Beleg und ändert keine Punkte.
- Zwei abgeschlossene Partien betreffen denselben Spieler: Locks serialisieren Poolupdates; stabile Reihenfolge vermeidet zyklische Sperren.
- Resultat ist Draw durch Wiederholung: normaler Score 0,5; Abort ist kein Draw.
- Cancel und Pairing kommen gleichzeitig: Actorreihenfolge entscheidet, nur ein persistenter Abschluss.
- Queueclient verliert Verbindung: kein automatisch erfolgreicher Cancel; explizite Abfrage nach Reconnect.
- Spieler erhält zweiten Login: aktives Ticket bleibt am Principal, nicht am Socket.
- Junqi verwendet verdeckte Eigenregeln: keine stillschweigende Übernahme des Xiangqi-Ratingpools.
- Sehr unterschiedliche Ratings: Formel bleibt numerisch endlich; speicherbare Grenzen und Präzision sind vorab geprüft.

## 10. Testing Requirements

| Given | When | Then |
|---|---|---|
| Zwei Ratings 1500 | erster Spieler gewinnt | 1516 und 1484 bei K32 |
| Zwei Ratings 1500 | Draw | beide bleiben 1500 |
| Ein gewertetes Resultat | Worker wiederholt nach Crash | gleicher Beleg, keine zweite Änderung |
| Fehler zwischen zwei Ratingwrites | Transaktion scheitert | beide Ratings und Beleg unverändert |
| Gastpartie, späteres Upgrade | Worker prüft alte Partie | weiterhin unrated |
| Ticketqueued | Cancel wird vor Pairing verarbeitet | cancelled und kein neuer Seat |
| Pairing committed | Cancel trifft danach ein | paired mit derselben `game_id` |
| Basiszeiten 300 und 301 Sekunden | Pool gewählt | blitz beziehungsweise rapid |
| Zwei Enqueues desselben Principals | parallel | höchstens ein aktives Ticket |
| Serverrestart | Client fragt altes Ticket | eindeutiger abgebrochener Queueplatz oder persistentes Pairing |

Rechenfälle sind reine deterministische Tests; Transaktions- und Lockfälle benötigen PostgreSQL, Queuefenster eine virtuelle Zeitquelle.

## 11. Definition of Done

- [ ] Ratingalgorithmus, Präzision und Poolzuordnung sind versioniert dokumentiert.
- [ ] Jeder bewertete Abschluss verändert beide Ratings atomar und einmalig.
- [ ] Abort, Gäste, Regelvarianten und alte Gastpartien sind sicher ausgeschlossen.
- [ ] Cancel-/Pairing-Races haben eindeutige getestete Ergebnisse.
- [ ] Crash-Recovery benötigt keine flüchtige Warteschlange als einzige Wahrheit.
- [ ] Profil und Leaderboard zeigen Poolbezug; mathematische und Integrationsnachweise liegen vor.

## 12. Common Pitfalls

Rating ist keine Synchronisationsbarriere für das Spielende. Wer den Game Actor bis zum gesamten Leaderboardupdate blockiert, koppelt Verfügbarkeit und Spielabwicklung unnötig.

Ein `processed`-Boolean nach zwei separaten Ratingwrites erlaubt halbe oder doppelte Wertungen. Ein eindeutiger Beleg wirkt nur, wenn er in derselben Transaktion wie beide Änderungen geschrieben wird. Ein Matchmaking-Cancel kann nach erfolgtem Pairing nicht rückwirkend zwei bereits vergebene Sitze verschwinden lassen.

## 13. Reflection Questions

1. Welche zusätzlichen Aussagen über Unsicherheit kann Glicko-2 gegenüber einfachem Elo ausdrücken?
2. Warum müssen beide Ratingänderungen aus denselben alten Werten berechnet werden?
3. Wie entsteht ein einmaliger Effekt trotz mehrfach gestarteter Worker?
4. Warum ist die Ratingfähigkeit eine Eigenschaft bei Spielbeginn und nicht des heutigen Accountstatus?
5. Welche Antwort ist korrekt, wenn Cancel nach committed Pairing kommt?
6. Was müsste sich ändern, bevor zwei Serverinstanzen dieselbe Matchmakingqueue besitzen dürfen?

## 14. Further Reading

- [Glickman: Glicko-2](https://www.glicko.net/glicko/glicko2.pdf): Rating Deviation, Volatilität und Ratingperioden als Designvergleich studieren; Originalpapier am 07.11.2026 abgerufen. Keine vollständige Algorithmusimplementierung aus diesem Kapitel ableiten.
- [FIDE Handbook](https://handbook.fide.com/): Rating Regulations als Vergleich für erwarteten Score und Policyunterschiede lesen; Kiban behauptet keine FIDE-Kompatibilität.
- [PostgreSQL Explicit Locking](https://www.postgresql.org/docs/current/explicit-locking.html): Zeilensperren und konsistente Sperrreihenfolge auf zwei Ratingzeilen anwenden.
- [Datenbanktransaktionen](../database/transactions.md): persistente Ergebnisverarbeitung und eindeutige Ratingbelege abgleichen.
- [Concurrency-Tests](../testing/concurrency-tests.md): Cancel-/Pairing- und doppelte Abschlussfälle erweitern.

## 15. Completion Record

- Abschlussdatum: ____________________
- Getroffene Designentscheidungen und Begründungen: ____________________
- Aufgetretene Schwierigkeiten und ihre Ursachen: ____________________
- Durchgeführte Tests und überprüfbare Ergebnisse: ____________________
- Verbleibende technische Schulden mit Priorität: ____________________
- Gelernte Zusammenhänge und nächster Lernschritt: ____________________
