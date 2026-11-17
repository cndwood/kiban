# REST-API

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Semantik

HTTP GET ist nebenwirkungsfrei, POST ist ohne request_id nicht automatisch idempotent. Alle Antworten verwenden application/json außer204. Geschützte Ressourcen verlangen Cookie+CSRF im Browser oder Bearer nativ. Pflichtfelder sind streng typisiert; unbekannte eingehende Mutationsfelder werden abgelehnt, damit Schreibfehler nicht still wirken. Request-Korrelation und Fach-request_id sind getrennt.

| Methode und Pfad | Eingabe | Erfolg | Wichtige Fehler |
|---|---|---|---|
| POST /api/v1/guest-sessions | display_name |201 Principal und Ablauf; nativ Token einmal, Browser Set-Cookie |400,429 |
| POST /api/v1/session/revoke | aktive Session, CSRF bei Cookie |204, erneuter Logout ebenfalls204 |401 bei fremden Credentials |
| POST /api/v1/games | request_id, game_kind, ruleset_id, visibility, time_control |201 GameView und ggf invitation_code |400,401,409,429,503 |
| GET /api/v1/games | cursor, limit1..50, game_kind |200 öffentliche waiting Spiele |400,429 |
| POST /api/v1/games/{game_id}/join | request_id, expected_version, ggf invitation_code |200 neue GameView |401,404,409,503 |
| POST /api/v1/games/{game_id}/cancel | request_id, expected_version |200 cancelled View, nur eigener waiting Seat |401,404,409 |
| GET /api/v1/games/{game_id} | Session und View-Recht |200 aktuelle/abgeschlossene Projektion |401,404 |
| GET /api/v1/games/{game_id}/moves | cursor, limit |200 berechtigte History-Seite |401,404 |
| GET /health/live | keine |200 Prozess antwortet |503 |
| GET /health/ready | interne Betriebsabfrage |200 Repository und Recovery bereit |503 |

Create-Idempotency: Scope Principal+request_id+create_game, Payload-Hash; Originalantwort samt Einladung nur berechtigt erneut liefern, verschlüsselte kurzfristige Response-Ablage statt Klartextlogging. invitation_code Hash allein kann Original nicht rekonstruieren: die verschlüsselte Retry-Antwort wird mit Ablauf oder Aktivierung entfernt; gewählt wird eine verschlüsselte Receipt-Antwort bis Aktivierung, Schlüssel außerhalb DB. Join ist atomar, Einladung verfällt mit Aktivierung.

## Beispiele

```json
{"request_id":"create-A","game_kind":"xiangqi","ruleset_id":"kiban-xiangqi-casual-v1","visibility":"public","time_control":{"initial_ms":180000,"increment_ms":2000}}
```

```json
{
  "game": {
    "game_id": "game-A",
    "state_version": 1,
    "status": "waiting",
    "visibility": "public",
    "game_kind": "xiangqi",
    "ruleset_id": "kiban-xiangqi-casual-v1",
    "seats": [
      {
        "side": "red",
        "principal_id": "principal-A",
        "display_name": "Gast A"
      }
    ],
    "position": {
      "side_to_move": "red",
      "pieces": [
        {
          "side": "red",
          "kind": "chariot",
          "x": 0,
          "y": 0
        },
        {
          "side": "red",
          "kind": "horse",
          "x": 1,
          "y": 0
        },
        {
          "side": "red",
          "kind": "elephant",
          "x": 2,
          "y": 0
        },
        {
          "side": "red",
          "kind": "advisor",
          "x": 3,
          "y": 0
        },
        {
          "side": "red",
          "kind": "general",
          "x": 4,
          "y": 0
        },
        {
          "side": "red",
          "kind": "advisor",
          "x": 5,
          "y": 0
        },
        {
          "side": "red",
          "kind": "elephant",
          "x": 6,
          "y": 0
        },
        {
          "side": "red",
          "kind": "horse",
          "x": 7,
          "y": 0
        },
        {
          "side": "red",
          "kind": "chariot",
          "x": 8,
          "y": 0
        },
        {
          "side": "red",
          "kind": "cannon",
          "x": 1,
          "y": 2
        },
        {
          "side": "red",
          "kind": "cannon",
          "x": 7,
          "y": 2
        },
        {
          "side": "red",
          "kind": "soldier",
          "x": 0,
          "y": 3
        },
        {
          "side": "red",
          "kind": "soldier",
          "x": 2,
          "y": 3
        },
        {
          "side": "red",
          "kind": "soldier",
          "x": 4,
          "y": 3
        },
        {
          "side": "red",
          "kind": "soldier",
          "x": 6,
          "y": 3
        },
        {
          "side": "red",
          "kind": "soldier",
          "x": 8,
          "y": 3
        },
        {
          "side": "black",
          "kind": "chariot",
          "x": 0,
          "y": 9
        },
        {
          "side": "black",
          "kind": "horse",
          "x": 1,
          "y": 9
        },
        {
          "side": "black",
          "kind": "elephant",
          "x": 2,
          "y": 9
        },
        {
          "side": "black",
          "kind": "advisor",
          "x": 3,
          "y": 9
        },
        {
          "side": "black",
          "kind": "general",
          "x": 4,
          "y": 9
        },
        {
          "side": "black",
          "kind": "advisor",
          "x": 5,
          "y": 9
        },
        {
          "side": "black",
          "kind": "elephant",
          "x": 6,
          "y": 9
        },
        {
          "side": "black",
          "kind": "horse",
          "x": 7,
          "y": 9
        },
        {
          "side": "black",
          "kind": "chariot",
          "x": 8,
          "y": 9
        },
        {
          "side": "black",
          "kind": "cannon",
          "x": 1,
          "y": 7
        },
        {
          "side": "black",
          "kind": "cannon",
          "x": 7,
          "y": 7
        },
        {
          "side": "black",
          "kind": "soldier",
          "x": 0,
          "y": 6
        },
        {
          "side": "black",
          "kind": "soldier",
          "x": 2,
          "y": 6
        },
        {
          "side": "black",
          "kind": "soldier",
          "x": 4,
          "y": 6
        },
        {
          "side": "black",
          "kind": "soldier",
          "x": 6,
          "y": 6
        },
        {
          "side": "black",
          "kind": "soldier",
          "x": 8,
          "y": 6
        }
      ]
    },
    "clock": null,
    "result": null
  },
  "invitation_code": null
}
```

Listencursor ist opak und bildet stabile Sortierung created_at+game_id ab. Liste zeigt keine Tokens, Einladungen oder private Games. Bei Join-Konflikt kein automatisches neues Spiel erzeugen. Offline-Clients testen diese Verträge vor Godot. Accounts/Matchmaking-Endpunkte sind Erweiterungen ab Kapitel27/28 und dürfen MVP-Verträge nicht umdefinieren.

## Erweiterungsverträge Kapitel27–29

Diese Endpunkte werden erst in den genannten Kapiteln implementiert. Mutationen verwenden request_id im Principal+Operation-Scope beziehungsweise Ticket-Scope; keine Passwörter/Recoverycodes in gespeicherten Klartext-Receipts. Nach unklarem Credential-Commit wird über Login mit dem neuen Passwort aufgeklärt. Session-/Recovery-Secrets werden einmal ausgegeben; ein Retry mit bereits committed Upgrade liefert account_created und Loginhinweis, keine erneute geheime Ausgabe.

| Methode und Pfad | Eingaben | Ergebnis und Rechte |
|---|---|---|
| POST /api/v1/accounts/upgrade | Gastsession, request_id, handle, password |201 gleicher principal_id; rotierte Session, einmalige Recoverycodes |
| POST /api/v1/sessions/login | handle,password |201 neue Session, generische401 bei falschen Credentials |
| GET /api/v1/sessions | aktuelle Accountsession |200 ausschließlich eigene Session-IDs, Ablauf und Gerätebezeichnung, keine Tokens |
| POST /api/v1/sessions/{session_id}/revoke | eigene Accountsession, request_id |204 nur eigene Session widerrufen |
| POST /api/v1/accounts/password | reauth password, new_password, request_id |200 rotierte Session über normalen Credentialkanal; andere Sessions widerrufen |
| POST /api/v1/accounts/recover | handle,recovery_code,new_password,request_id |204 Code atomar verbraucht, alle Sessions widerrufen; anschließend Login |
| POST /api/v1/accounts/recovery-codes | aktuelle Session plus password,request_id |200 neue Codes einmal, alte Codes ungültig |
| GET /api/v1/profiles/{principal_id} | öffentliche ID |200 Anzeigename, Statistiken, Poolratings; keine Loginmetadaten |
| POST /api/v1/profile | aktuelle Accountsession,request_id,display_name |200 eigene öffentliche Projektion |
| POST /api/v1/matchmaking/tickets | Accountsession,request_id,game_kind,ruleset_id,time_control |201 ticket_id,status queued; nur unterstützte rated Xiangqi-Pools |
| GET /api/v1/matchmaking/tickets/{ticket_id} | Eigentümersession |200 queued/cancelled/paired mit game_id nur wenn paired |
| POST /api/v1/matchmaking/tickets/{ticket_id}/cancel | Eigentümersession,request_id |200 cancelled oder bereits paired, niemals beides |
| GET /api/v1/leaderboards | pool_id,cursor,limit1..50 |200 paginierte öffentliche Profile/Ratings |
| GET /api/v1/games/{game_id}/rating | berechtigte View |200 pending/applied/not_applicable, nur berechtigte öffentliche Werte |
| GET /api/v1/games/{game_id}/replay | berechtigte Session,cursor,limit1..50 |200 nach Version geordnete berechtigte historische Views |

Zuschauer öffnen öffentliche Games über subscribe_game; Rolle wird serverseitig aus Seatbelegung abgeleitet, niemals aus einer vertrauenswürdigen Clientbehauptung. Private Games sind ausschließlich für Teilnehmer sichtbar, auch nach Ende. Replay verwendet dieselben Zugriffsregeln. Live-Xiangqi-Zuschauen ist im ersten Umfang unverzögert; externe Hilfe/Collusion wird damit nicht verhindert und ist kein gelöstes Anti-Cheating-Problem.

Loginhandle: ASCII a–z,0–9,_ mit3–24 Zeichen, Eingabe ASCII-kleingeschrieben; keine Verwechslung mit Unicode-Anzeigenamen. Passwort12–128 Unicode-Zeichen, maximal512 UTF-8-Bytes, unverändert hashen, nicht trimmen oder normalisieren. Argon2id-Parameter anhand aktueller OWASP-Empfehlung und Zielhardware benchmarken und als versioniertes Profil speichern. Recoverycodes:10 unabhängige128Bit-Zufallswerte, nur Hashes und Verbrauchszeit speichern, unbenutzte bis Regeneration gültig. Recovery- und Loginlimits werden gesondert pro IP/Handle plus globalem Hashbudget getestet.

Elo:1500 initial, K32; Pools pro game_kind+ruleset_id+Basiszeitbucket, <=300000ms blitz, darüber rapid. Intern sechs Dezimalstellen, Änderung einmal auf sechs Stellen runden und der Gegenseite exakt negiert geben; Anzeige auf nächste ganze Zahl runden, .5 weg von null. Ein Ergebnisworker pro Prozess verarbeitet nach finished_at,game_id; beide alten Werte derselben Transaktion nutzen. Public Leaderboard zeigt nur Accounts mit mindestens einer rated Partie, Tiebreak PrincipalId; „vorläufig“ bis20 gewerteten Spielen ist ein Anzeigehinweis, ändert K nicht.

Matchmaking verwendet zunächst festes ±100-Fenster, genaue gleiche Zeitkontrolle/Profil und ältestes kompatibles Ticket mit ticket_id als Tiebreak. Kapitel28 beschreibt optional eine Erweiterung; diese bleibt Experiment ohne Änderung der Basisabnahme. Nicht gepaarte Tickets werden beim Restart cancelled(server_restart); gepaarte bleiben abfragbar und referenzieren dieselbe Partie (die unter Restart-Policy aborted sein kann).

GET /api/v1/games listet ausschließlich öffentliche waiting-Partien mit freiem Seat; volle Junqi-Deployments bleiben waiting, sind aber nicht beitretbar. GET /api/v1/games/history mit eigener Session, Cursor und limit1..50 liefert die eigene Historienliste (öffentliche und eigene private Spiele); fremde private Spiele bleiben ausgeschlossen. Dieser feste Listenpfad wird vor dem parametrischen games/{game_id}-Handler geroutet.

Create-Response ist ein Wrapper mit game als vollständiger GameView und invitation_code als String ausschließlich für den Ersteller privater Spiele, sonst null. Join/Cancel liefern denselben Wrapper ohne neues Geheimnis (invitation_code=null). Create-TimeControl besitzt initial_ms/increment_ms; optionale rated-Anforderung ist zunächst false und erst ab Kapitel28 freigeschaltet. rated=true erfordert registrierten Ersteller und registrierten zweiten Spieler bei Aktivierung; Gast-Create/Join wird dafür explizit abgewiesen statt still umgedeutet. Matchmaking erzeugt ausschließlich rated=true für kompatible Accounttickets. GameView wird ab Kapitel28 additiv um rated und pool_id erweitert, laufende Altspiele behalten rated=false.
