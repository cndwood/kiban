# WebSocket-Nachrichten

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Lebenszyklus

Nach HTTPS-Upgrade ist kein Spielcommand erlaubt, bevor authenticate erfolgreich war. Native Clients senden Token im ersten Frame; Browser verwenden die sichere beim Upgrade geprüfte Cookie-Session und payload ohne Token. authenticate ohne Authnachweis wird geschlossen. authentication_ok meldet Principal und Ablauf, ohne Credential. Danach subscribe_game oder resync; Actor prüft View-Rechte und registriert Snapshot+Stream atomar.

| type | Richtung | Bedeutung |
|---|---|---|
| authenticate | Client→Server | Session innerhalb5s bestätigen |
| authentication_ok | Server→Client | Principal-Verbindung ist authentifiziert |
| subscribe_game | Client→Server | game_id abonnieren, initialer Snapshot |
| unsubscribe_game | Client→Server | Subscription entfernen |
| resync | Client→Server | vollständigen Snapshot samt neuem Stream-Cursor beziehen |
| make_move | Client→Server | Spielaktion mit request_id und expected_version |
| resign | Client→Server | Aufgabe mit denselben Mutationsfeldern |
| snapshot | Server→Client | vollständige berechtigte View auf Version V |
| game_event | Server→Client | event_kind plus vollständige berechtigte View auf V+1 |
| command_ack | Server→Client | bestätigtes oder dupliziertes Mutationsergebnis |
| error | Server→Client | strukturierter Fehler ohne Zustandswechsel |
| heartbeat | beide Richtungen | Lebenszeichen ohne state_version-Erhöhung |

submit_deployment und ready_deployment kommen ausschließlich für Junqi ab Kapitel30 hinzu. make_move trägt dort from/to-Graphkoordinaten; Ränge sendet der Client nur im eigenen Deployment, nie als behauptetes Combat-Ergebnis.

```json
{"protocol_version":1,"type":"snapshot","game_id":"game-A","state_version":8,"payload":{"status":"active","game_kind":"xiangqi","ruleset_id":"kiban-xiangqi-casual-v1","position":{"side_to_move":"black","pieces":[{"side":"red","kind":"general","x":4,"y":0},{"side":"black","kind":"general","x":4,"y":9},{"side":"red","kind":"soldier","x":4,"y":5}]},"clock":{"remaining_ms":{"red":178000,"black":180000},"active_side":"black","sampled_at_server_ms":1794052800000},"result":null,"game_id":"game-A","state_version":8,"visibility":"public","seats":[{"side":"red","principal_id":"principal-A","display_name":"Gast A"},{"side":"black","principal_id":"principal-B","display_name":"Gast B"}]}}
```

Das Beispiel zeigt eine reduzierte zulässige Testposition, keine vollständige Startaufstellung. Piece-Liste hat eindeutige Koordinaten; die folgenden Beispiele enthalten alle GameView-Pflichtfelder und Sitzinformationen, ohne geheime Sessiondaten. Clock.remaining_ms ist zum Snapshot-Abtastzeitpunkt berechnet; sampled_at_server_ms ist nur Diagnose, keine Clientautorität.

```json
{"protocol_version":1,"type":"game_event","game_id":"game-A","state_version":9,"payload":{"event_kind":"game_finished","state":{"status":"finished","game_kind":"xiangqi","ruleset_id":"kiban-xiangqi-casual-v1","position":{"side_to_move":"black","pieces":[{"side":"red","kind":"general","x":4,"y":0},{"side":"black","kind":"general","x":4,"y":9},{"side":"red","kind":"soldier","x":4,"y":5}]},"clock":{"remaining_ms":{"red":178000,"black":179000},"active_side":null,"sampled_at_server_ms":1794052801000},"result":{"winner":"red","reason":"resignation","final_version":9},"game_id":"game-A","state_version":9,"visibility":"public","seats":[{"side":"red","principal_id":"principal-A","display_name":"Gast A"},{"side":"black","principal_id":"principal-B","display_name":"Gast B"}]}}}
```

Snapshot und game_event verwenden denselben vollständigen Spielzustand; game_event trägt ihn unter payload.state, snapshot direkt unter payload. Identität und Version werden in Envelope und GameView identisch geführt; Sichtbarkeit und Sitze sind GameView-Pflichtfelder. Beide Beispiele zeigen dieselbe reduzierte Testposition.

## Fehler und Trennung

Acks und Events können bei Retry relativ zueinander eintreffen. Client ordnet Zustand nur über state_version, Pending Absicht über request_id. Overflow schließt langsame Verbindung mit erklärendem Resync-Hinweis; kein endloses Puffern. Schließen eines Sockets bedeutet weder Aufgabe noch Uhrpause.

## Exakte lesende und Session-Envelopes

authenticate trägt keine game_id/expected_version; payload ist nativ {"token":"einmaliger-geheimer-Sessionwert"}, im Cookie-Modus {}. authentication_ok.payload enthält principal_id und expires_at (RFC3339 UTC). subscribe_game, unsubscribe_game und resync tragen protocol_version,type,game_id,request_id und payload={}; sie sind lesende Control Requests ohne expected_version und erzeugen kein Mutation-Receipt. Snapshot trägt zur Antwortkorrelation request_id des Control Requests. game_event besitzt kein Client-request_id, weil es auch systemseitig ausgelöst sein kann. heartbeat trägt protocol_version,type und payload={} und kein fachliches Versionsfeld. resign verwendet die normale Mutations-Envelope mit payload={}.

submit_deployment enthält payload.pieces als Array aus25 Objekten mit piece_id,kind,x,y; serververgebene Piece-IDs und das eigene vollständige Multiset werden geprüft. ready_deployment enthält payload={}. Gegnerische Ränge kommen in keiner dieser Aktionen vor. make_move enthält nur from/to, keinen Client-Combat-Ausgang. Deployments als Änderungen erhöhen state_version; deren Events enthalten pro Empfänger gefilterte Views.
