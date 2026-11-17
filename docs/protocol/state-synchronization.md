# State Synchronization

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Vollständige Views statt Patchsprache

GameView enthält game_id, game_kind, ruleset_id, state_version, status, visibility, autorisierte Sitzdaten, spielabhängige position, clock oder null und result oder null. Snapshot-Envelope und GameView-Version müssen übereinstimmen. Der Actor ist einziger Snapshot-Erzeuger für laufende Spiele; abgeschlossene Views werden aus Repository und demselben Projektor aufgebaut.

Subscriber-Registration und Snapshot-Erzeugung sind ein serieller Actor-Turn. Nach Snapshot V wird Event V+1 oder später geordnet geliefert. Erreicht ein Client V+2 ohne V+1, sperrt er Mutation und fordert resync. Gleiche/ältere Version ist Duplicate und wird verworfen. Ein unveränderter Snapshot darf dieselbe Version mit aktuelleren präsentationalen Clock-Restwerten tragen; das verändert keine dauerhafte Uhrbasis.

## Wiederholung und Autorisierung

Receipt-Lookup hat Scope game_id+principal_id+request_id und vergleicht kanonischen Command-Hash einschließlich Type, Ziel und expected_version. Gleicher Command liefert Originalantwort, selbst wenn current_version höher ist. Gleiche ID mit anderem Command liefert request_id_conflict. Pro Principal und Partie höchstens eine offene Mutation im Client, trotzdem muss Server Mehrgeräte-Races korrekt abweisen.

Ein Reconnect mit Einladung allein erhält keinen Spielerblick. Bei Junqi wird view_for(principal/role) vor Serialisierung aufgerufen, auch für History, Fehlerdetails, Ack und Debugpfade. Auf keine unbekannten Ränge kann ein Client „zur Optimierung“ zugreifen.

## Abbruchfälle

Wenn Ack verloren, erneut identischer Request. Wenn Broadcast verloren, Snapshot. Wenn Commit unklar, recovery_required und DB-Abgleich. Wenn Prozess neu, aktive Partie aborted(server_restart) mit höherer dauerhafter Version; veralteter Client darf sie nicht reaktivieren. Das Versionsprotokoll verspricht geordnete Zustände, keine globale Ordnung zwischen verschiedenen Spielen.

## Verbindliches GameView-Schema v1

| Feld | Form und Bedeutung |
|---|---|
| game_id | opaker String, gleich Envelope.game_id |
| state_version | Integer0..2^53−1, gleich Envelope.state_version |
| game_kind | xiangqi oder ab Kapitel30 junqi |
| ruleset_id | unveränderlicher veröffentlichter Profilstring |
| status | waiting, active, finished, cancelled oder aborted |
| visibility | public oder private |
| seats | Array mit höchstens2 Objekten: side red/black, principal_id String, display_name String; keine Sessiondaten |
| position | spielabhängiges Objekt, bei noch nicht aktiver Xiangqi-Partie Startposition; Junqi Deploymentview gemäß Rolle |
| clock | null für untimed/waiting oder Objekt remaining_ms mit red/black Integern, active_side red/black/null, sampled_at_server_ms Integer |
| result | null oder Objekt winner red/black/null, reason stabiler Code, final_version Integer |

Xiangqi position besitzt side_to_move und pieces; jedes Piece besitzt side, kind (general,advisor,elephant,horse,chariot,cannon,soldier), x und y. Koordinaten sind eindeutig. result bei finished trägt sportliche Gründe checkmate, stalemate, resignation, timeout, repetition oder perpetual_check; aborted trägt server_restart oder persistence_failure mit winner=null. cancelled trägt cancellation oder waiting_expired, ebenfalls winner=null. final_version ist die Version des Endübergangs.

Junqi position besitzt phase (deployment/playing/ended), side_to_move oder null, own_ready, opponent_ready, ply, pieces und captured. Ein Piece besitzt opake rangunabhängige piece_id, side, x,y und known_kind; known_kind ist null, wenn diese Identität für den Empfänger verborgen ist. captured enthält nur bereits öffentliche Kampfdaten mit piece_id,side,known_kind und event_version. Eigene noch veränderbare Deploymentposition wird ausschließlich an den Eigentümer ausgegeben; vor Aktivierung sieht Gegner keine Belegungs-/Rangdaten. Zuschauer sehen ausschließlich öffentliche readiness. own_ready/opponent_ready sind bei neutralem Zuschauer null; öffentliche readiness wird zusätzlich über red_ready/black_ready geführt. Alle Junqi-Views enthalten diese vier Readyfelder konsistent, und nach Aktivierung dieselben öffentlichen Positionen mit berechtigtem known_kind. Ergebnisse ergänzen flag_captured, no_legal_moves und ply_limit; Pilot unrated, clock=null.

Interne Daten und optionale spätere Anzeigefelder dürfen dieses Pflichtschema nur nach den Regeln der Protokollversionierung erweitern. Ein komplettes Golden Fixture enthält alle Pflichtfelder, keine Ellipsen oder nachträglich zu ergänzenden Platzhalter. Seat-Principals in öffentlichen Views sind pseudonyme IDs, keine Loginnamen oder Kontaktdaten.
