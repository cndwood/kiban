# Kapitel 21 — Lobby UI, Spielbrowser und Einladungen

[Dokumentationsindex](../INDEX.md) · [Projektfortschritt](../checklists/project-progress.md)

## 1. Chapter Overview

Du verbindest die bereits getesteten REST-Operationen mit einer bedienbaren Godot-Lobby. Das Ergebnis ist ein Ablauf von einer Gastidentität über die Auswahl eines öffentlichen Spiels oder einer privaten Einladung bis zum richtigen Spielbildschirm. Noch wird kein Zug durch UI-Code entschieden.

Eine Lobby ist eine zeitlich begrenzte Sicht auf serverseitige Daten. Der Bildschirm muss deshalb erklären können, dass ein angezeigter Platz beim Klick schon vergeben ist. Ein sauber behandelter Konflikt ist erwartetes Verhalten, kein Absturz.

## 2. Prerequisites

- Kapitel 01–20 abgeschlossen; besonders Gastidentität aus Kapitel 10, Lobbykontrakte aus Kapitel 11 und der Netzwerkdienst aus Kapitel 20.
- Godot-Szenen, Signals, Controls, Anchors und die Trennung von Netzwerkdienst und View Model sind bekannt.
- Vor der Arbeit die [REST-Verträge](../protocol/rest-api.md) und [Frontendgrenzen](../architecture/godot-frontend.md) lesen.
- Für lokale Tests existieren ein laufendes Backend und zwei getrennte Gastidentitäten. Statische JSON-Fixtures bleiben als schnelle UI-Testalternative verfügbar.

## 3. Learning Objectives

- Der Entwickler kann Ladezustand, leere Trefferliste, Berechtigungsfehler und Transportfehler sichtbar unterscheiden.
- Er kann konkurrierende Join-Versuche demonstrieren, ohne einen lokalen Sitzplatz als bestätigt darzustellen.
- Er kann einen veralteten Browser-Response gezielt verwerfen und erklären, welches Versionsmerkmal dies erlaubt.
- Er kann öffentliche und private Spiele durch dieselbe serverseitige Autorisierung führen.

## 4. Concepts and Theory

### UI State Machine

**Definition:** Eine UI State Machine beschreibt zulässige Zustände wie `loading`, `ready`, `submitting` und `failed` samt Übergängen. **Motivation:** Mehrere unabhängige Booleans erlauben widersprüchliche Anzeigen. **Einsatz:** Das Formular hat während einer laufenden Erstellung genau einen Submission-Zustand. **Fehler:** Ein disabled Button wird als alleiniger Schutz gegen doppelte Erstellung betrachtet. **Geeignet:** Wenn Netzwerkantworten verzögert eintreffen und Benutzer weiterhin navigieren. **Vertiefung:** Godots Dokumentation zu Signals in Abschnitt 14 mit den serverseitigen States aus Kapitel 02 vergleichen.

### Stale Responses und Cancellation

**Definition:** Eine Response ist stale, wenn sie zu einer älteren Filterauswahl oder einem verlassenen Bildschirm gehört. **Motivation:** Netzwerkreihenfolge entspricht nicht zwingend der Reihenfolge von Klicks. **Einsatz:** Jede Browserabfrage erhält lokal eine Generation; nur die aktuelle Generation darf das View Model ersetzen. **Fehler:** Cancellation des sichtbaren Requests wird mit serverseitigem Rückgängigmachen verwechselt. **Geeignet:** Suche, Pagination und Szenenwechsel. **Vertiefung:** HTTPRequest-Lebenszyklus und Signals in den Godot-Referenzen.

### Idempotency an der Bedienoberfläche

**Definition:** Ein wiederholter logischer Auftrag hat denselben beabsichtigten Effekt. **Motivation:** Ein verlorener Response beweist nicht, dass die Erstellung scheiterte. **Einsatz:** Ein Wiederholungsversuch benutzt dieselbe `request_id`; eine bewusst neue Partie bekommt eine neue ID. **Fehler:** Jeder Timeout erzeugt automatisch eine neue ID und damit eine zweite Partie. **Geeignet:** Mutierende REST-Aktionen und spätere Spielkommandos. **Vertiefung:** [Wiederverbindung](../architecture/reconnection.md).

### Accessible Feedback

**Definition:** Zustandsänderungen werden durch lesbaren Text, Fokusführung und konsistente Interaktion verständlich. **Motivation:** Eine rote Farbe allein erklärt keinen Fehler. **Einsatz:** Einladungsfehler erscheinen am Feld, Serverprobleme am Bildschirm. **Fehler:** Fokus nach jedem Refresh zurücksetzen. **Geeignet:** Alle Formulare und Listen. **Vertiefung:** Godots GUI-Tutorials und Control-Referenz.

## 5. Architectural Context

`LobbyScreen` besitzt ausschließlich Anzeigezustand und kurzlebige Eingaben. `LobbyViewModel` übersetzt fachliche Antworten in sichtbare Zustände. Der vorhandene Netzwerkdienst besitzt Verbindung, Session und Requestkorrelation. Lobbydaten bleiben im Backend; die UI reserviert keine Plätze.

Datenfluss: Eingabe → validierter Formularauftrag → REST-Service → Serverantwort → View Model → Signals → Controls. Bei erfolgreichem Join wird die bestätigte `game_id` an die Navigation übergeben; anschließend startet der schon vorhandene Snapshot-Abonnementfluss. Die laufende Spiellogik bleibt im Game Actor.

## 6. Design Decisions

**Verbindlich:** Öffentliche Spiele werden seitenweise per REST abgefragt. Ein serverseitiger Push-Browser wäre schneller aktuell, würde aber ein weiteres langlebiges Abonnement einführen. Polling bei aktivem Bildschirm ist für das MVP verständlich und ausreichend; Backoff bei Fehlern verhindert Lastspitzen. Bei gemessener Lobbylast diese Entscheidung neu bewerten.

**Verbindlich:** Private Einladungen sind keine Session-Credentials. Der Code sucht ein Spiel; ein authentifizierter Join vergibt den Sitz. Ein Link allein darf weder Identität noch Spielerfarbe festlegen. Das ist etwas mehr UI-Arbeit als direktes Öffnen eines Sockets, erhält aber die Autorisierungsgrenze.

**Optional:** Lokale Sortierung innerhalb einer geladenen Seite; eine globale Sortierreihenfolge muss später serverseitig spezifiziert werden.

## 7. Implementation Tasks

- [ ] 1. Lege im Lobby-View-Model die Zustände `loading`, `ready`, `empty`, `submitting` und `failed` fest. Zeichne nach Kapitel 18 ihre erlaubten Übergänge; jede sichtbare Control-Kombination muss einem Zustand zuordenbar sein.
- [ ] 2. Baue eine statische Spieleliste mit Fixtures aus Kapitel 20. Zeige Spieltyp, Regelprofil, Zeitkontrolle und freie Plätze; prüfe lange IDs und schmale Fenster ohne Netzwerk.
- [ ] 3. Verbinde die Listenabfrage mit dem REST-Service. Übernimm nur die neueste lokale Abfragegeneration; belege dies mit zwei absichtlich vertauschten Responses.
- [ ] 4. Ergänze Pagination im View Model. Verwende den Servercursor unverändert und setze ihn bei Filterwechsel zurück; prüfe erste, letzte und leere Seite.
- [ ] 5. Baue das Erstellungsformular mit den angebotenen Zeitkontrollen 1+0, 3+2, 5+0 und 10+5. Lokale Validierung verbessert Feedback, der Server prüft alle Werte erneut.
- [ ] 6. Ergänze öffentliche und private Sichtbarkeit. Nach erfolgreicher privater Erstellung zeige den Einladungscode als explizit kopierbaren Text; prüfe, dass er nicht im öffentlichen Browser erscheint.
- [ ] 7. Führe die Erstellung mit einer pro logischem Auftrag stabilen `request_id` aus. Simuliere einen verlorenen Response und überprüfe, dass Retry dieselbe Partie öffnet.
- [ ] 8. Ergänze Join je Listenzeile. Deaktiviere nur die betroffene Aktion während Submission; bei Konflikt aktualisiere die Liste und erläutere den vergebenen Platz.
- [ ] 9. Baue den Einladungscode-Einstieg über den spezifizierten privaten Join. Entferne nur erlaubte Rand-Leerzeichen; prüfe ungültiges Format, unbekannten und abgelaufenen Code ohne Informationsleck.
- [ ] 10. Navigiere erst nach bestätigtem Join mit der Server-`game_id` zum Spiel. Lasse einen manipulierten lokalen Farbwert wirkungslos und prüfe die Farbe aus dem Snapshot.
- [ ] 11. Behandle abgelaufene Gastsession als eigenen Zustand im Session-Service. Verlange eine bewusste neue Session statt still die alte Spieleridentität auszutauschen; prüfe Rückkehr zur Lobby.
- [ ] 12. Entferne beim Szenenwechsel Signals und UI-Abonnements. Ein verzögerter HTTP-Response darf die bereits freigegebene Szene nicht ansprechen; verifiziere wiederholtes Öffnen und Schließen.
- [ ] 13. Ergänze Fokusreihenfolge, Tastaturbedienung und Fehlertext. Prüfe das komplette Erstellen und Beitreten ohne Maus sowie bei hohem UI-Scale.
- [ ] 14. Führe mit zwei Clients den gesamten Lobbyablauf durch. Bewahre Request- und Game-IDs als Testnachweis auf, niemals Session-Tokens.

## 8. Public Interfaces and Contracts

| Grenze | Eingaben | Ergebnis und Fehler |
|---|---|---|
| `LobbyViewModel.refresh` | Filter, Cursor, lokale Generation | aktuelle Seite oder sichtbarer Ladefehler; stale Response ohne Mutation |
| `create_game` im REST-Service | `request_id`, `game_kind`, `ruleset_id`, Zeitkontrolle, Sichtbarkeit | bestätigte Partie; private Antwort nur berechtigten Empfängern |
| `join_game` | Session, `game_id` oder Einladungscode laut REST-Vertrag | serverseitiger Sitz oder Konflikt; keine freie Farbauswahl durch Clients |
| Navigation | bestätigte `game_id` | Spielszene; kein Autorisierungsnachweis |

Die genaue Wire-Form ist in [REST API](../protocol/rest-api.md) kanonisch. Die UI erfindet keine alternativen Endpunkte. Fehlercodes steuern die Darstellung; menschenlesbare Servertexte werden als Text und niemals als Markup ausgeführt.

## 9. Edge Cases

- Ein Spiel verschwindet zwischen Listenabfrage und Join: Konflikt erklären und neu laden.
- Zwei Klicks oder ein Responseverlust: denselben Auftrag wiederholen, kein zweites Spiel erfinden.
- Eine private Einladung wird weitergereicht: Sitzvergabe bleibt serverseitig und kann bereits abgeschlossen sein.
- Nach einem Filterwechsel kommt die vorherige Seite später an: Generation verhindert Überschreiben.
- Szene wird während eines Requests geschlossen: Response beendet den Request, verändert aber keine fremde Szene.
- Token läuft während der Erstellung ab: keine automatische neue Identität und kein blindes erneutes Erstellen.

## 10. Testing Requirements

| Given | When | Then |
|---|---|---|
| Leere öffentliche Lobby | erster Refresh gelingt | erklärter Leerzustand und Erstellen-Aktion sichtbar |
| Zwei freie-Spiele-Responses A und B | B wird vor A zugestellt | ausschließlich B bleibt sichtbar |
| Ein letzter freier Sitz | zwei Clients treten gleichzeitig bei | genau einer navigiert als neuer Spieler, der andere erhält Konflikt |
| Privates Spiel | öffentlicher Browser wird geladen | weder Code noch privater Datensatz erscheint |
| Erstellung committed, Response verloren | derselbe Auftrag wird wiederholt | dieselbe `game_id`, kein Duplikat |
| Laufender Refresh | Szene wird verlassen | keine Mutation oder Fehlermeldung in der nächsten Szene |
| Ungültige Zeitkontrolle | Request wird trotz UI-Prüfung manipuliert | Server lehnt ab und Formular bleibt korrigierbar |

Nutze steuerbare Fake-Responses für Reihenfolge und Timeouts; nur der abschließende Vertragstest benötigt einen echten Server.

## 11. Definition of Done

- [ ] Öffentliche und private Erstellung sowie Join sind mit zwei Identitäten demonstriert.
- [ ] Alle definierten UI-Zustände und jeder Fehlerpfad haben einen Fixture-Test.
- [ ] Stale Responses, Doppelclicks und Szenenwechsel sind deterministisch geprüft.
- [ ] Keine lokalen UI-Felder gelten als Sitz- oder Identitätsnachweis.
- [ ] Netzwerkdetails bleiben im Service; Signals werden beim Verlassen aufgeräumt.
- [ ] Bedienung, Screenshots der Fehlerzustände und Integrationsnachweise sind dokumentiert.

## 12. Common Pitfalls

Ein Spinner ohne Abbruch- oder Fehlermöglichkeit hält Benutzer dauerhaft fest. Ein globales `busy` blockiert oft unbeteiligte Listenelemente und verschleiert den tatsächlichen Auftrag. Ein Auto-Refresh, der die Liste vollständig neu baut, zerstört Scrollposition und Tastaturfokus.

Eine private URL ist keine Zugriffskontrolle. Prüft der Client allein das Codeformat, können direkte HTTP-Aufrufe die Beschränkung umgehen. Behandle jeden Erfolg erst nach Serverbestätigung als Erfolg.

## 13. Reflection Questions

1. Wieso kann ein bereits sichtbarer freier Platz beim Join fehlen, obwohl beide Systeme korrekt arbeiten?
2. Welche Informationen muss eine lokale Generation enthalten, die eine `request_id` nicht enthält?
3. Wann bedeutet Wiederholen denselben Auftrag und wann eine neue Benutzerabsicht?
4. Weshalb darf eine verlorene Session nicht unbemerkt durch eine neue Gastidentität ersetzt werden?
5. Welche Lobbyzustände sollten bei fehlender Internetverbindung noch bedienbar bleiben?
6. Wie würdest du globale Sortierung und cursorbasierte Pagination miteinander vereinbaren?

## 14. Further Reading

- [Godot: UI-System](https://docs.godotengine.org/en/stable/tutorials/ui/index.html): Containers, Fokus und Skalierung für Listen und Formulare studieren.
- [Godot: HTTPRequest](https://docs.godotengine.org/en/stable/classes/class_httprequest.html): Requestabschluss und Fehlerklassen mit dem Netzwerkdienst abgleichen.
- [Godot: Signals](https://docs.godotengine.org/en/stable/getting_started/step_by_step/signals.html): Lebensdauer von Verbindungen bei Szenenwechsel prüfen.
- [REST-Verträge](../protocol/rest-api.md): alle Formularfelder vor der Anbindung gegen das kanonische Schema prüfen.

## 15. Completion Record

- Abschlussdatum: ____________________
- Getroffene Designentscheidungen und Begründungen: ____________________
- Aufgetretene Schwierigkeiten und ihre Ursachen: ____________________
- Durchgeführte Tests und überprüfbare Ergebnisse: ____________________
- Verbleibende technische Schulden mit Priorität: ____________________
- Gelernte Zusammenhänge und nächster Lernschritt: ____________________
