# Repetition, Check und Chase

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Casual-Algorithmus als Vertrag

Zweck: endlose identische Zyklen deterministisch auflösen, ohne komplexe Turnier-Schiedsrichterlogik vorzutäuschen. Eingabe: initiale Position und lückenlose Folge akzeptierter Positionen mit side_to_move und Flag, ob der gerade ausgeführte Zug den Gegner schachsetzt. Ausgabe: laufend, Remis oder Niederlage des alleinigen Dauerschachgebers. Keine Netzwerkzeit und keine Clock-Budgets gehen in Positionsgleichheit ein.

Positionsgleichheit bedeutet identische Farbe+Art auf allen90 Schnittpunkten und gleiches Zugrecht. Hash ist nur Index; tatsächliche Gleichheit muss Hashkollisionen auflösen. Initialposition zählt als erstes Auftreten. Beim dritten Auftreten derselben Position werden die Züge ausschließlich zwischen zweitem und drittem Auftreten als letzter abgeschlossener Zyklus betrachtet. Eine Seite qualifiziert, wenn sie in diesem Zyklus mindestens einen Zug hatte und jeder ihrer Züge Schach gab. Qualifiziert genau eine Seite, verliert diese; qualifizieren beide oder keine, repetition. Diese Entscheidung ist absichtlich enger und einfacher als offizielle Check-/Chase-Urteile.

Speicher wächst O(P) für P akzeptierte Halbzüge, naive Suche O(P) je Zug. Eine Hash→Vorkommensliste reduziert erwartete Suche, muss aber Gleichheit und ausreichende Check-History bewahren. Undo/Replay und Crash Recovery müssen dieselbe Sequenz reproduzieren. Move History allein ohne Ausgangsposition und ruleset_id reicht nicht.

## Abgrenzung zum Turnier

Perpetual Chase bedeutet wiederholte konkrete Schlagdrohung/Verfolgung, nicht jedes beliebige Attack-Flag. Offizielle Regeln unterscheiden geschützte Figuren, Austausch, Gegenangriffe und weitere Kategorien. Kiban-Casual wertet Chase nicht automatisch: auch ein Chase-Zyklus kann nach obigem Profil Remis werden. WXF2018 enthält umfangreiche illustrierte Fälle; deren vollständige Übernahme ist ein eigener Arbeitsschritt mit Quellenprüfung und neuer ruleset_id, keine einzelne zusätzliche if-Abfrage.

## Sequenztests

- Position tritt zweimal auf: kein Ergebnis.
- Drittes Auftreten, gleicher Boardinhalt aber anderes Zugrecht: kein Treffer.
- Zyklus beide Seiten ohne Check: Remis.
- Rot checkt auf jedem eigenen Zykluszug, Schwarz mindestens einmal nicht: Rot verliert.
- Beide Seiten checken auf jedem eigenen Zykluszug: Remis im Casual-Profil.
- Rot checkt zuerst, aber nicht auf jedem eigenen Zykluszug: kein unilateral-check-Verlust.
- Gleicher Hash, unterschiedlicher Boardinhalt: kein falsches Ergebnis.
- Restart-History enthält alle früheren Auftreten; Replay reproduziert die Entscheidung, aktive Restart-Partie wird dennoch abgebrochen.

Sequenztests dürfen synthetische geprüfte Position-Key-/Check-Daten für den Repetition-Baustein nutzen. Ein Integrationstest muss zusätzlich eine vollständig legale Zugfolge nachweisen. Der Lernende dokumentiert konkrete Koordinaten, lässt jeden Zug durch die Legalitäts-API laufen und prüft die letzte Zyklusabgrenzung.
