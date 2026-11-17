# Dokumentationsvalidierung

[Index](INDEX.md) · [Fortsetzungsstand](WORK_REMAINING.md)

Prüfdatum:07.11.2026. Geprüft wird das Handbuch, keine Anwendung. Die zukünftigen Programmtests werden hier nicht als ausgeführt behauptet.

## Umfang und Ergebnis

| Prüfung | Ergebnis |
|---|---|
| Pflichtstruktur gegen MASTER_PROMPT.md |93 von93 erforderlichen Markdown-Dateien vorhanden |
| Gesamter neuer Dokumentationssatz |96 Markdown-Dateien einschließlich README, Quellenregister, Validierung und Fortsetzungsstand |
| Kapitel |30 von30, jeweils exakt15 geforderte Abschnitte in richtiger Reihenfolge |
| Implementation Tasks |502 insgesamt; je Kapitel12–24 konkrete Checkbox-Aufgaben |
| Reflexionsfragen |je Kapitel6–8, ohne vorweggenommene Antworten |
| Relative Links |676 relative Links; alle Ziele und vorhandenen Anker auflösbar |
| Erreichbarkeit |alle Pflichtdateien vom INDEX aus erreichbar |
| JSON-Beispiele |alle7 JSON-Codeblöcke syntaktisch parsebar |
| Mermaid |alle36 Blöcke durch Mermaid12.3.0 erfolgreich geparst |
| Verbotene Implementierungen |keine Rust-/GDScript-/SQL-/YAML-/TOML-/Dockerfile-Implementierungsblöcke; Repository-Dateien ausschließlich Markdown |
| Projektinitialisierung |kein Cargo-Workspace, kein Godot-Projekt und kein Anwendungsskelett angelegt |

Das Handbuch umfasst rund74.000 Wörter einschließlich Navigation und Checklisten. Maßgeblich sind Inhalte und Nachweise, nicht diese Näherung.

## Methode

Ein dokumentationsbezogener Python-Prüflauf außerhalb des Repositories verglich eine explizite93-Dateien-Liste aus dem Master-Auftrag mit dem Dateisystem. Er prüfte Abschnittsnamen/-folge, Kapitelanzahl, Aufgaben-/Fragenanzahl, ausgeglichene Code-Fences, JSON mittels Standardparser, relative Linkziele/Anker und Erreichbarkeit per Graphdurchlauf ab INDEX. Er kontrollierte außerdem auf nicht-Markdown-Dateien außerhalb der vorhandenen Git-Metadaten und auf verbotene Codeblocksprachen. Zusätzliche Suchläufe prüften TODO/TBD, widersprüchliche Bezeichner und alte Einheiten.

Für Mermaid wurde die echte Parserbibliothek12.3.0 mit DOM-Umgebung ausschließlich unter /tmp installiert und jeder extrahierte Block einzeln geparst. Keine Parser-/Generatorabhängigkeit wurde dem Kiban-Repository hinzugefügt. Die Diagramme wurden syntaktisch geprüft; dies ist kein Screenshot- oder Layouttest in allen Markdown-Renderern.

## Inhaltlicher Abgleich

Drei parallele Kapitelstränge wurden gegen gemeinsame Verträge abgestimmt und anschließend quergeprüft. Korrigiert wurden PrincipalId statt PlayerId, initial_ms/increment_ms statt gemischter Einheiten, final_version und repetition als konsistente Ergebnisnamen, die spezielle idempotente Logoutbehandlung, Timeout-before-Command-Receipts und ein ungültiger Cannon-Testausgang. GameView wurde mit Pflichtfeldern und vollständigen JSON-Beispielen festgelegt. Account-/Matchmaking-/History-Endpunkte und zugehörige DB-Entitäten wurden ergänzt.

Verbindliche Architektur bleibt: modularer Monolith, getrennte Engines, ein Actor je Partie, bounded Queues, Autorität auf dem Server, monotone Deadline bei Actor-Bearbeitungsbeginn, Commit vor Ack/Broadcast, durable Receipts und neutraler Abbruch aktiver Restart-Partien. Temporärer Speicherbetrieb vor Kapitel17 ist klar als nicht durable markiert. Godot wird zuerst mit statischen Fixtures und Multiplayer zuerst mit Terminalclients geprüft. Junqi besitzt eigenen Graphen, Deployment und empfängerabhängige Views statt einer wiederverwendeten Xiangqi-Matrix.

## Abdeckung der16 geforderten Diagrammthemen

| Thema | Nachweis |
|---|---|
| Gesamtsystem |[Overview](architecture/overview.md) |
| Workspace-Abhängigkeiten |[Rust-Workspace](architecture/rust-workspace.md) |
| Xiangqi-Komponenten |[Engine-Grenzen](architecture/rust-workspace.md) |
| Game Lifecycle |[Domain Model](architecture/domain-model.md) |
| Gastsession |[Authentication](architecture/authentication.md) |
| HTTP-Erstellung |[Backend](architecture/backend.md) |
| WS-Lifecycle |[Networking](architecture/networking.md) |
| Move Validation |[Move-Sequenz](architecture/backend.md) |
| Actor-Nachrichtenfluss |[Actors](architecture/actor-model.md) |
| Clock-Übergänge |[Uhren](architecture/chess-clocks.md) |
| Timeout-/Move-Race |[Deadline-Sequenz](architecture/chess-clocks.md) |
| Reconnect |[Wiederverbindung](architecture/reconnection.md) |
| DB-Beziehungen |[Schema](database/schema.md) |
| Crash Recovery |[Recovery](database/recovery.md) |
| Godot-Navigation |[Frontend](architecture/godot-frontend.md) |
| Junqi-Views |[Multigame](architecture/multigame.md) |

## Quellen und Grenzen

Das [Quellenregister](SOURCES.md) benennt tatsächlich per Netzwerk geprüfte offizielle Einstiegsseiten und beobachtete Versionen. Nicht jeder externe Unterseitenlink wurde erneut angefragt; die Linkprüfung hier gilt vollständig für relative lokale Links. Bewegliche stable/latest-Dokumentation ersetzt kein späteres Dependency-Pinning.

Xiangqi verwendet kiban-xiangqi-casual-v1 mit bewusst vereinfachter Repetition, ohne automatische Perpetual-Chase-Schiedslogik. Es ist kein Turnierprofil. Junqi verwendet das vollständig spezifizierte eigene Hidden-Rank-Profil kiban-junqi-open-v1, kein behauptetes Verbandsregelwerk. Die Testszenarien sind Spezifikationen für spätere selbst geschriebene Tests, kein bereits bewiesener Regelengine-Betrieb.

Offene Dokumentationsaufgaben: keine. Geplante spätere Implementierungsentscheidungen und bewusste Grenzen stehen im Fortsetzungsstand sowie den jeweiligen ADRs/Kapiteln.
