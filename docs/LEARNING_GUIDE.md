# Lernleitfaden

[Index](INDEX.md)

Arbeite die 30 Kapitel in Reihenfolge durch. Plane mehrere Monate; schwierige Regeln, Recovery und Concurrency benötigen wiederholte Experimente. Vorkenntnisse: Variablen, Funktionen, Datenstrukturen, Git-Grundidee; Rust wird ab Kapitel 01 aufgebaut. Ein Linux-, macOS- oder Windows-System mit Terminal und später Docker sowie Godot genügt.

Pro Kapitel: zunächst Voraussetzungen prüfen, Konzepte in eigenen Worten erklären, Entscheidungen schriftlich festhalten, kleine Aufgaben einzeln implementieren, die Given/When/Then-Fälle ausführen und erst danach Definition of Done abhaken. Der Completion Record ist ein Arbeitsjournal und darf nicht als bestandener Test gelten. Bei Problemen zuerst den minimalen reproduzierbaren Fall, die offizielle API-Dokumentation und eigene Tests nutzen; keine KI-generierten Implementierungen übernehmen.

Die Dokumente nennen Schnittstellen und erwartetes Verhalten, liefern aber keine fertigen Lösungen. Befehle sind Lernhinweise für die spätere Eigenarbeit. In dieser Dokumentationsphase wird kein Projekt initialisiert.

## Integrationsgrenzen

1. Kapitel 01–08: deterministische Engine und Terminaladapter, kein Netzwerk und keine Datenbank.
2. Kapitel 09–13: HTTP/WebSocket und Speicher-Repositories; zwei Terminalclients demonstrieren Multiplayer. Ein Prozessabbruch darf in dieser ausdrücklich temporären Stufe Zustand verlieren.
3. Kapitel 14–16: Fake Clock und Reconnect; noch keine Crash-Durability versprechen.
4. Kapitel 17: PostgreSQL ersetzt Speicher-Repositories, Commit vor Bestätigung wird verpflichtend.
5. Kapitel 18–22: Godot zuerst mit statischen Fixtures, dann gegen dasselbe Protokoll.
6. Kapitel 23–26: Fehlerfälle, Sicherheit, Deployment, Betriebsnachweise.
7. Kapitel 27–30: Accounts, Rating, Zuschauer und zweite Engine; keine Plattformkopie.

Prüfe nach jedem Meilenstein den manuellen Nachweis. Bei fehlendem Vorwissen darf eine Lerneinheit wiederholt werden; spätere Infrastruktur soll frühere Fachlogik nicht verdecken. Architekturänderungen beginnen mit einer ADR und einer Liste betroffener Verträge, Tests und Kapitel.
