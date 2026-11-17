# Ergebnis und Endspiel

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Ergebnispriorität

Transport-/Auth-/Receipt-Prüfung zuerst, dann im aktiven Actor Deadline-Prüfung. Ist sie erreicht, gewinnt die Gegenseite durch timeout unabhängig von hypothetischem Matt. Vor der Deadline: zulässiger Resign endet durch resignation; legaler Zug wird angewandt, danach Antwortzüge geprüft, anschließend Repetition. Kein legaler Antwortzug mit bedrohtem General ist checkmate, ohne Bedrohung stalemate; beide sind Niederlage der am Zug befindlichen Seite. Erst wenn Antworten existieren, darf Casual-Repetition ein Ergebnis setzen.

GameResult enthält winner red/black oder null bei draw, reason und final_version. finished benötigt Result; aborted(server_restart/persistence_failure) ist neutral und für Ratings ausgeschlossen. Inkrement wird nur nach nichtterminalem Zug addiert. Ein Result wird genau einmal committed; weitere Commands liefern invalid_state oder ihr bereits bekanntes Receipt.

## Warum nicht westliches Schach?

Patt ist hier kein Remis. Zwei Generäle dürfen einander nicht offen gegenüberstehen. Kein materielles Heuristikremis wird vom MVP geraten: selbst wenn ein Mensch eine Stellung als theoretisch remis betrachtet, folgt daraus keine automatische Engineentscheidung. Eine spätere Zuggrenzenregel müsste Zählweise, Resets, Checks und Priorität mit eigenem Profil spezifizieren.

## Testbare Positionen

Ausführbare Fixtures stehen in [Regeltests](rule-test-positions.md). Besonders wichtig: schwarze Generalstellung(4,9), rote Chariots(3,8),(5,8), roter General(4,0), roter Soldier(4,5), Schwarz am Zug ist Patt und Schwarz verliert. Ergänze roten Soldier(4,8): Schwarz steht im Schach und hat keine legale Antwort, also Matt. Entferne einen flankierenden Chariot und prüfe die neu mögliche Flucht statt nur das Endlabel.

Ungültige Testpositionen mit fehlendem General, doppelten Figurenkoordinaten oder offen stehenden Generälen sind Validierungsfehler, keine Siege. Testbuilder darf für reine Bewegungsprüfung reduzierte Bretter erlauben; Full-Game-Tests benötigen vollständige Generalinvarianten.
