# Deterministische Uhrtests

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

## Testabstraktion

Clock liefert monotone now-Werte; TestClock wird explizit fortgeschaltet. Scheduler/Deadline-Wakeup ist getrennt vom Zeitwert, damit verspätetes Wakeup prüfbar bleibt. Alle Tests laufen ohne reale Sekundenwartezeit. Ausgangswerte und Ergebnisbudgets sind Integer-Millisekunden.

| Given | When | Then |
|---|---|---|
|1000ms Rest, Start0 |Move bei999 |kein Timeout, legaler nichtterminaler Zug bekommt Inkrement |
|1000ms Rest |Move bei1000 |Timeout gewinnt, kein Move |
|1000ms Rest |Move bei1001 |gleiches Ergebnis |
|1000ms, Inkrement2000 |legaler nichtterminaler Zug bei400 |2600ms für alte Seite |
|1000ms, Inkrement2000 |terminaler legaler Zug bei400 |600ms, kein Inkrement |
|1000ms Rest |illegaler Zug bei400, später legal bei800 |Gesamtverbrauch800, kein Reset |
|aktiver Zug, Deadline1000 |Timer wacht erst1500 auf |Timeout abgelaufen, nicht zusätzliche500ms geschenkt |
|Turnwechsel |alter Timer für vorige Generation |kein falscher Timeout |
|Ack verloren |identischer Retry |kein zweites Inkrement |
|Systemzeit springt±1h |gleiche monotone Werte |identische Entscheidung |
|Disconnect bei400 |Gegner wartet bis1000 |Timeout, keine Pause |
|zwei Timeout-Anstöße |beide verarbeitet |ein Resultat und ein terminales Event |

Zusätzlich DB-Commitverzögerung: Gegnerdeadline bleibt am erfassten Entscheidungspunkt verankert; lange Commitzeit zählt zur neuen Seite. Bei Persistenzfehler keine weitere Spielannahme, Abbruch nach Reconciliation. Snapshot-Clock ist zum Sende-Abtastwert normalisiert; Anzeige darf driften, Resultat nicht. Restart active wird aborted(server_restart), nicht willkürlich timeout aus UTC.
