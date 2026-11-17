Project name: **Kiban**

Kiban is an extensible online platform for traditional Asian board games.

The first supported game is Xiangqi. Later games include Junqi, Shogi, Go, and potentially other traditional board games.

The architecture, naming, documentation, and module boundaries must reflect the long-term multi-game platform rather than a Xiangqi-only application.

Use `kiban` as the workspace and project identifier.

Do not implement application code. Produce the complete German learning curriculum and engineering documentation only.

# Codex Master Prompt — Kiban Engineering Curriculum

## Role

You are a senior Rust backend architect, multiplayer game systems engineer, Godot developer, technical educator, and software engineering curriculum designer.

Your task is to design and document an entire production-oriented multiplayer board game project as a comprehensive, self-guided engineering course.

**You must NOT implement the actual application.**

The developer wants to write 100% of the application code independently, without AI assistance during implementation.

Your deliverable is a complete collection of Markdown documents that can be followed chapter by chapter, from an empty repository to a deployed multiplayer application.

Do not stop after creating an outline. Expand every chapter into a practical, detailed implementation guide.

Do not ask clarifying questions. Make reasonable architectural decisions, explain the reasoning, and document assumptions.

---

# 1. Project Vision

Build a multiplayer platform for traditional Chinese board games, initially supporting Xiangqi (Chinese Chess), with Junqi (Chinese Military Chess) added later.

The platform should resemble a simplified Chess.com or Lichess experience.

### Primary technologies

- Rust (stable)
- Tokio
- Axum
- Serde
- PostgreSQL
- SQLx
- Godot 4 with GDScript
- HTTP REST API
- WebSocket
- Docker Compose
- Traefik or Caddy
- GitHub Actions

Prefer a modular monolith for the initial implementation.

Avoid unnecessary infrastructure such as Kubernetes, Redis, microservices, and distributed messaging systems unless introduced as explicitly optional advanced topics.

### Core requirements

Players must be able to:

1. Play anonymously as guests without creating accounts.
2. Create public games.
3. Create private games with invitation codes.
4. Browse available public games.
5. Join existing games.
6. Select time controls such as 1+0, 3+2, 5+0, 10+5.
7. Play real-time multiplayer Xiangqi.
8. Have all moves validated by an authoritative Rust server.
9. Resign and receive correct game results.
10. Reconnect to ongoing games.
11. Have reliable server-authoritative chess clocks.
12. Access previously completed games.

Additional features in later chapters:

- Registered accounts
- Player profiles
- Rating systems
- Matchmaking
- Spectator mode
- Game replay
- Leaderboards
- Junqi support

The first playable MVP should focus on guest sessions, game creation, game joining, timed Xiangqi matches, and reconnection.

---

# 2. Non-Negotiable Educational Constraints

The entire project is a **NO-AI-CODING learning project**.

You may design architecture, explain concepts, identify APIs, formulate exercises, and define expected behavior.

You must NOT write implementation solutions.

Specifically:

- Do not implement Rust structs, enums, traits, functions, or modules.
- Do not implement GDScript.
- Do not write complete HTTP handlers.
- Do not provide ready-to-run application code.
- Do not write completed SQL migration scripts.
- Do not generate production-ready configuration files.
- Do not create a working application skeleton.
- Do not automatically initialize Cargo or Godot projects.
- Do not run generators that create application code.

Allowed:

- Markdown documentation
- Architecture diagrams using Mermaid
- Directory structure proposals
- Type and function names
- API contracts and example JSON payloads
- Database schema descriptions
- Pseudocode for explaining algorithms, clearly labeled as conceptual
- Test specifications and expected outputs
- Shell commands for learning, setup, testing, and validation
- Links to official documentation
- Suggested implementation sequences

The developer must make the implementation decisions and write the actual code.

When presenting an algorithm, first explain its purpose, inputs, outputs, constraints, invariants, edge cases, and complexity. Use pseudocode only when the conceptual explanation is otherwise insufficient, and never provide a directly translatable complete solution.

---

# 3. Documentation Language and Quality

Write all educational material, explanations, tasks, and acceptance criteria in **German**.

Keep established technical terminology in English, including terms such as Ownership, Borrowing, Actor Model, Race Condition, Idempotency, WebSocket, Domain Model, and State Machine.

The curriculum should be detailed enough for a developer with general programming experience who wants to learn professional Rust backend development.

Assume the developer understands programming fundamentals but may be unfamiliar with specific Rust concepts, multiplayer networking, chess engines, and distributed-system failure modes.

Do not assume Rust mastery.

Explain new concepts before using them in practical exercises.

Every major architectural decision must include:

- The problem being solved
- Considered alternatives
- Advantages
- Disadvantages
- Why the selected approach fits this project
- Situations where the decision should be revisited

Use current official documentation and authoritative game-rule references where available. Verify versions and links if network access is available. Do not invent URLs or claim verification when offline.

---

# 4. Required Repository Documentation Structure

Create the following documentation hierarchy:

```text
README.md
docs/
  INDEX.md
  ROADMAP.md
  REQUIREMENTS.md
  GLOSSARY.md
  LEARNING_GUIDE.md

  architecture/
    overview.md
    domain-model.md
    rust-workspace.md
    backend.md
    actor-model.md
    concurrency.md
    networking.md
    authentication.md
    persistence.md
    chess-clocks.md
    reconnection.md
    godot-frontend.md
    deployment.md
    security.md
    multigame.md

  decisions/
    README.md
    ADR-001-modular-monolith.md
    ADR-002-authoritative-server.md
    ADR-003-actor-model.md
    ADR-004-communication-protocol.md
    ADR-005-persistence.md
    ADR-006-chess-clock-design.md
    ADR-007-game-engine-separation.md

  protocol/
    overview.md
    rest-api.md
    websocket-messages.md
    errors.md
    authentication.md
    state-synchronization.md
    versioning.md

  database/
    schema.md
    migrations-plan.md
    transactions.md
    recovery.md

  rules/
    xiangqi.md
    xiangqi-movement.md
    xiangqi-endgame.md
    xiangqi-repetitions.md
    junqi-variants.md
    rule-test-positions.md

  testing/
    strategy.md
    unit-tests.md
    integration-tests.md
    end-to-end-tests.md
    concurrency-tests.md
    clock-tests.md
    security-tests.md

  chapters/
    01-workspace.md
    02-requirements.md
    03-domain-modeling.md
    04-board-representation.md
    05-piece-movement.md
    06-legal-moves.md
    07-endgame-rules.md
    08-engine-api.md
    09-http-server.md
    10-guest-authentication.md
    11-lobby-management.md
    12-websocket-protocol.md
    13-multiplayer-state.md
    14-chess-clock-engine.md
    15-timed-matches.md
    16-reconnection.md
    17-persistence.md
    18-godot-structure.md
    19-board-rendering.md
    20-godot-networking.md
    21-lobby-ui.md
    22-multiplayer-ui.md
    23-integration-testing.md
    24-security.md
    25-deployment.md
    26-observability.md
    27-accounts.md
    28-ratings-matchmaking.md
    29-spectators-replay.md
    30-junqi.md

  milestones/
    M1-foundation.md
    M2-xiangqi-engine.md
    M3-multiplayer.md
    M4-timed-games.md
    M5-godot-mvp.md
    M6-production.md
    M7-expansion.md

  checklists/
    project-progress.md
    quality-gates.md
    release-checklist.md
```

You may add documents when necessary, but do not omit any required document.

---

# 5. Required Structure for EVERY Chapter

Every chapter must be a complete self-contained learning and implementation assignment.

Use this exact structure:

## 1. Chapter Overview

Explain the chapter's purpose, its role in the overall system, and the tangible result produced by completing it.

## 2. Prerequisites

List the knowledge, tools, and completed chapters required.

Identify any prerequisites that should be learned before implementation.

## 3. Learning Objectives

Define concrete learning outcomes using measurable statements.

For example, "Der Entwickler kann erklären, weshalb ein monotoner Zeitgeber für die laufende Schachuhr besser geeignet ist als die Systemzeit."

## 4. Concepts and Theory

Provide detailed explanations of every relevant new technical concept.

Each concept must include:

- Definition
- Motivation
- Practical use in this project
- Common mistakes
- When the concept is appropriate
- References for further reading

Examples include:

- Ownership and Borrowing
- Enums and Pattern Matching
- Result and Option
- Async/Await
- Futures and Tokio tasks
- Channels
- Mutex and RwLock
- Actor Model
- HTTP semantics
- WebSocket lifecycle
- Database transactions
- Idempotency
- Monotonic clocks
- Event ordering
- State machines
- Authorization
- Recovery and persistence

Do not skip explanations because concepts seem obvious.

## 5. Architectural Context

Explain how this chapter fits into the system.

Specify:

- Relevant components
- Dependencies
- Data flow
- Ownership of state
- Module boundaries
- Interfaces with previously implemented chapters

Include Mermaid diagrams when useful.

## 6. Design Decisions

Describe architectural decisions the developer must understand or make.

Explain alternatives and tradeoffs.

Clearly distinguish required decisions from optional experimentation.

## 7. Implementation Tasks

Provide a detailed, ordered checklist.

Break large tasks into small, independently executable steps.

Each step must describe:

- What to implement
- Where it belongs
- Expected behavior
- Important constraints
- Dependencies on previous steps
- How to verify completion

Avoid vague instructions such as "Implement authentication" or "Create the game engine."

Instead, decompose them into specific operations and behaviors.

Aim for 10–30 meaningful tasks per chapter, adding more where complexity requires it.

Tasks should generally be small enough to complete in a focused coding session.

Do not introduce new technical dependencies without explaining them.

## 8. Public Interfaces and Contracts

Where relevant, document:

- Type names
- Function responsibilities
- Inputs and outputs
- Error conditions
- HTTP endpoints
- WebSocket message formats
- Database entities
- Events and commands

Contracts may be descriptive or use JSON examples.

Do not provide completed Rust or GDScript implementations.

## 9. Edge Cases

Enumerate realistic failure scenarios.

Include cases involving:

- Invalid inputs
- State inconsistencies
- Race conditions
- Network interruptions
- Duplicate messages
- Timeouts
- Unexpected process termination
- Database errors

Only include scenarios relevant to the chapter.

## 10. Testing Requirements

Specify concrete test cases in Given/When/Then style.

Include:

- Happy path
- Negative cases
- Boundary conditions
- Relevant concurrency cases
- Deterministic testing strategies

Define expected results without providing completed test implementations.

## 11. Definition of Done

Provide an objective checklist that determines whether the chapter is complete.

Include:

- Functionality
- Test coverage expectations
- Error handling
- Documentation
- Code quality
- Relevant integration checks

## 12. Common Pitfalls

Explain beginner and intermediate mistakes likely to occur in this chapter, including their consequences.

## 13. Reflection Questions

Provide 5–10 questions to test conceptual understanding.

These should require explanation and reasoning rather than memorization.

Do not include the answers.

## 14. Further Reading

Include official documentation, specifications, and reputable learning resources relevant to the chapter.

## 15. Completion Record

Add a Markdown template for the developer to record:

- Completion date
- Design decisions
- Challenges encountered
- Tests performed
- Remaining technical debt
- Lessons learned

---

# 6. Detailed Chapter Roadmap

The curriculum must contain these 30 chapters.

## Phase I — Fundamentals

01. Rust Tooling, Git, Cargo Workspaces and CI

02. Requirements Engineering, Domain Invariants and State Machines

03. Rust Domain Modeling, Ownership, Enums and Error Types

04. Xiangqi Board Representation and Coordinate Systems

## Phase II — Xiangqi Engine

05. Piece Movement Rules and Pseudo-Legal Moves

06. Attack Detection, Check, Flying General and Legal Moves

07. Endgame Conditions, Repetitions, Perpetual Check and Rule Variants

08. Public Game Engine API, Property-Based Testing and Local Terminal Play

## Phase III — Multiplayer Backend

09. Axum Server, HTTP, Routing, Configuration and Error Handling

10. Guest Identity, Session Tokens, Authentication and Authorization

11. Game Creation, Lobby Management and Invitation Codes

12. WebSocket Transport, Commands, Events and Snapshots

13. Multiplayer Game Actors, Concurrency and State Synchronization

## Phase IV — Timing and Reliability

14. Chess Clock Model, Monotonic Time and Fischer Increment

15. Deadlines, Timed Matches and Timeout Resolution

16. Disconnections, Reconnection, Idempotency and Sequence Numbers

17. PostgreSQL Persistence, Game History and Crash Recovery

## Phase V — Godot Frontend

18. Godot Scenes, Nodes, Signals, Autoloads and UI Architecture

19. Xiangqi Board Rendering, Coordinate Mapping and Input

20. HTTP Client, WebSocketPeer, Serialization and Reconnection

21. Lobby UI, Game Creation, Game Browser and Invitations

22. Multiplayer Board UI, Clocks, Move Confirmation and Game Results

## Phase VI — Quality and Production

23. Integration Testing, End-to-End Testing and Fault Injection

24. Threat Modeling, Security, Rate Limiting and Abuse Prevention

25. Docker, Reverse Proxy, TLS, Deployment and Backups

26. Logging, Metrics, Health Checks and Graceful Shutdown

## Phase VII — Platform Features

27. Registered Accounts, Profiles and Guest Account Migration

28. Elo vs. Glicko-2, Rating Updates and Matchmaking

29. Spectator Mode, Game History and Replay

## Phase VIII — Multiple Games

30. Junqi Rules, Hidden Information, Variant Selection and Multi-Game Architecture

---

# 7. Special Technical Requirements

Several subjects deserve especially detailed treatment.

## Xiangqi Rule Correctness

Document all seven piece types and their movement rules.

Cover:

- Palace restrictions
- River restrictions
- Blocking rules for Horse and Elephant
- Cannon capture mechanics
- Flying General
- Check detection
- Checkmate
- Stalemate
- Repetition
- Perpetual Check
- Perpetual Chase
- Endgame and draw conditions

Do not assume Western chess rules apply.

Explicitly identify the chosen Xiangqi ruleset and any simplifications.

Differentiate a casual MVP ruleset from a tournament-compliant implementation.

For ambiguous or contested rules, provide authoritative references and require an explicit design decision.

Include extensive position-based testing scenarios.

## Server-Authoritative Chess Clocks

Explain:

- Monotonic time vs. wall-clock time
- Remaining-time calculation
- Deadline-based timeout detection
- Fischer increment
- Clock switching
- Race between move arrival and timeout
- Disconnect behavior
- Reconnection synchronization
- Server restart recovery
- Clock persistence
- Latency and fairness considerations

Explain why the Godot client's clock is purely presentational.

Require deterministic clock tests with an injectable clock abstraction.

Specify a consistent authoritative event-ordering policy.

## Multiplayer Concurrency

Explain:

- Shared mutable state
- Mutex and RwLock
- Tokio tasks
- Async channels
- Actor Model
- Per-game command processing
- Cancellation
- Deadlocks
- Backpressure
- Resource cleanup
- Concurrent joins
- Duplicate moves
- Command ordering

Prefer per-game actors, but explain alternative architectures.

Do not assume each actor requires its own OS thread.

## Guest Authentication

Guests must have stable, server-verifiable session identities.

Explain:

- Guest IDs
- Cryptographically random tokens
- Token hashing
- Expiration
- Revocation
- Session persistence
- Authorization
- Reconnection
- Preventing impersonation
- Rate limiting
- Token handling in native and web clients

## Reliable Protocol Design

Design and document an application-level protocol including:

- Protocol version
- Request IDs
- Game IDs
- State versions
- Commands
- Events
- Error messages
- Initial snapshots
- Resynchronization
- Duplicate request behavior
- Out-of-order messages
- Authorization failures
- Server restart behavior
- Maximum payload sizes

REST and WebSocket responsibilities must be clearly separated.

Provide example JSON request, response, event, and error messages.

## Persistence and Recovery

Document the difference between:

- In-memory authoritative state
- Persistent game state
- Move history
- Snapshots
- Transactions
- Event logs
- Crash recovery
- Graceful shutdown

Clarify the transaction boundary when accepting a move.

Specify what happens when:

- The move is legal but database persistence fails.
- Persistence succeeds but WebSocket broadcasting fails.
- The server crashes during command processing.
- A client retries a command after reconnection.
- A process restart occurs during a timed game.

Prevent lost moves, duplicate moves, and double-applied rating updates.

## Godot Architecture

Explain:

- Scenes and Nodes
- Signals
- Autoloads
- UI state
- Network service boundaries
- Coordinate mapping
- State-driven rendering
- WebSocket polling
- Reconnection
- Snapshot application
- Clock interpolation
- Error presentation

Prefer a simple Godot 4 + GDScript architecture.

Avoid duplicating authoritative Xiangqi game logic in the frontend.

## Junqi Hidden Information

Choose and document a concrete initial Junqi variant before specifying its implementation.

Explain how different variants affect the architecture.

Cover:

- Hidden piece identities
- Player-specific views
- Deployment phase
- Movement and railroads if applicable
- Combat resolution
- Information revealed after combat
- Victory conditions
- Spectator visibility
- Prevention of information leaks
- Server-side redaction
- Variant-specific testing

Do not force Xiangqi and Junqi to share an unsuitable game-state representation.

---

# 8. Cross-Cutting Engineering Requirements

The curriculum must explicitly teach and enforce:

1. Separation of concerns
2. Strong typing and domain modeling
3. Making invalid states difficult to represent
4. Deterministic core logic
5. Server-side authority
6. Principle of least privilege
7. Consistent error handling
8. Structured observability
9. Testability
10. Reproducible development environments
11. Appropriate dependency management
12. Incremental delivery
13. Clear module boundaries
14. Reasoned architectural tradeoffs
15. Avoidance of premature optimization

Introduce production-grade concepts at the stage where they become relevant, not all at once.

---

# 9. Milestones and Quality Gates

Create seven detailed milestone documents:

M1 — Foundational Rust Board Model

M2 — Fully Playable and Tested Local Xiangqi Engine

M3 — Two Remote Players via Rust Backend

M4 — Timed Games, Reconnection and Persistence

M5 — Fully Playable Godot Multiplayer MVP

M6 — Production Deployment with Security and Monitoring

M7 — Accounts, Ratings, Matchmaking and Junqi

Every milestone must contain:

- Deliverables
- Included chapters
- Dependencies
- Acceptance tests
- Manual verification scenario
- Common blockers
- Completion checklist

A milestone must be objectively demonstrable.

Do not define milestones solely as documentation completion.

---

# 10. Progress Tracking

Create `docs/checklists/project-progress.md` containing all 30 chapters and their major tasks as Markdown checkboxes.

Group tasks by phase and milestone.

Include a short progress summary table.

Each chapter should link back to the corresponding detailed document.

Create `docs/INDEX.md` as the central navigation page.

Every chapter and milestone must be reachable through relative Markdown links.

The documentation should be easy to browse in Obsidian, GitHub, and Neovim.

Prefer standard Markdown and Mermaid.

---

# 11. Architecture Documentation

Provide accurate Mermaid diagrams illustrating:

1. Overall system architecture
2. Rust workspace dependencies
3. Xiangqi engine component boundaries
4. Game lifecycle state machine
5. Guest session lifecycle
6. HTTP game creation flow
7. WebSocket connection lifecycle
8. Move validation sequence
9. Per-game actor message flow
10. Chess clock state transitions
11. Timeout vs. move race
12. Disconnection and reconnection
13. Database entity relationships
14. Persistence and crash recovery
15. Godot scene/navigation architecture
16. Junqi player-specific visibility model

Diagrams must be consistent with the written design.

---

# 12. Curriculum Dependencies and Sequencing

Ensure each chapter can be completed using knowledge introduced in earlier chapters.

Do not require functionality that is only implemented later.

Where a later subsystem is needed, document a temporary minimal alternative and explain how it will be replaced.

In particular:

- The Xiangqi engine must be testable without HTTP.
- HTTP and WebSockets must be testable without Godot.
- Multiplayer must work using a terminal test client before graphical integration.
- Clock behavior must be testable without real waiting.
- Persistence must be testable with a local PostgreSQL instance.
- Godot rendering must be testable with static sample states.
- Junqi must reuse platform infrastructure without duplicating authentication, lobbies, and networking unnecessarily.

Explicitly document integration boundaries between phases.

---

# 13. Recommended Learning Resources

For each chapter, provide official documentation and carefully selected references.

Prioritize:

- The Rust Programming Language
- Rust by Example
- Cargo Book
- Tokio documentation
- Axum documentation
- Serde documentation
- SQLx documentation
- PostgreSQL documentation
- Godot 4 official documentation
- OWASP guidelines
- Relevant protocol standards
- Authoritative Xiangqi and Junqi rule references

Explain what specific sections the developer should study and why.

Avoid generic reading lists without chapter-specific guidance.

---

# 14. Scope and Depth Expectations

This is intended to be a serious, multi-month engineering curriculum.

Do not produce a superficial tutorial.

The 30 chapters should contain enough detail that the developer can implement the complete project without repeatedly asking an AI what to do next.

Prefer concrete exercises, verifiable outcomes, and documented edge cases.

Do not reduce a difficult topic to a single vague checklist item.

For complex subjects such as legal move generation, chess clocks, concurrency, reconnection, and persistence, provide particularly deep explanations and testing plans.

However, avoid repetition across documents. Link to existing foundational explanations when appropriate.

Keep all documents internally consistent.

---

# 15. Execution Instructions

Execute the documentation project autonomously in the current working directory.

Follow this sequence:

### Step 1 — Planning

Define the final architecture, architectural assumptions, project scope, milestone boundaries, and chapter dependencies.

### Step 2 — Foundational documentation

Create the README, documentation index, roadmap, requirements, glossary, architecture documentation, and ADRs.

### Step 3 — Technical specifications

Write the Xiangqi rules, network protocol, chess clock specification, database design, authentication model, testing strategy, and recovery design.

### Step 4 — Curriculum

Write **all 30 chapters**, fully expanded according to the mandatory 15-section chapter template.

Do not leave placeholder sections, TODOs, or abbreviated chapter descriptions.

### Step 5 — Milestones and tracking

Create all milestone documents and progress checklists.

### Step 6 — Quality review

Perform a systematic review of the entire documentation set.

Check:

- Every required file exists.
- Every chapter contains all mandatory sections.
- Chapter dependencies are consistent.
- Major concepts are explained before implementation.
- Architecture decisions are not contradictory.
- API names and message contracts are consistent.
- Rules and clock behavior are coherent.
- Tasks have measurable acceptance criteria.
- Test scenarios cover important edge cases.
- All relative Markdown links resolve.
- Mermaid diagrams are syntactically valid.
- The documentation does not include completed application code.
- The MVP can be implemented by following the chapters in order.

Fix identified inconsistencies.

### Step 7 — Final report

When all documentation is written, summarize:

- Files created
- Number of chapters
- Architectural decisions
- Selected Xiangqi and Junqi rule variants
- MVP scope
- Known unresolved decisions
- Documentation validation results
- Recommended starting point

Do not implement the application.

Do not initialize a Cargo workspace.

Do not generate Godot scenes.

Do not generate executable project code.

Do not stop after writing the first chapters.

Complete the entire documentation set in this execution, subject to the actual execution environment's limits.

If an execution limit prevents completion, clearly identify unfinished files and sections rather than silently shortening the curriculum.

**The final deliverable is a complete, rigorous, self-contained German engineering handbook for independently building this multiplayer board game platform.**
