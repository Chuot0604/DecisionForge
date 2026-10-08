Phase 0 – Projektbasis
│
├── Monorepo
│   ├── startlux/
│   ├── laya/
│   ├── api/
│   ├── experiments/
│   ├── planning/
│   └── infrastructure/
│
├── lokale GPU-Konfiguration
└── VPS-CPU-Konfiguration
          │
          ▼
Phase 1 – Decision Making verstehen
│
├── StartLux lokal starten
├── einfache Entscheidungen
├── Candidates / Options verstehen
├── Wahrscheinlichkeiten untersuchen
├── Prompt-Verhalten testen
└── Benchmarks Small vs. Large
          │
          ▼
Phase 2 – Decision API
│
├── POST /decision
├── Structured Input
├── Candidates
├── Scores
├── Top-K
├── Confidence
└── Logging
          │
          ▼
Phase 3 – Classification Wrapper API
│
├── Text
├── Dokumente
├── dynamische Kategorien
├── Batch Classification
├── Confidence Threshold
└── Structured Output
          │
          ▼
Phase 4 – Planner PoC
│
├── LLM
├── Ticket / MR
├── Kommentare
├── Dokumente
└── Structured Test Plan
          │
          ▼
Phase 5 – Playwright Executor
│
├── DOM / Accessibility Tree
├── Action Candidates
├── StartLux Decision
├── Execute
└── Observe
          │
          ▼
Phase 6 – Autonomous Testing
    ├── Goal Detection
    ├── Assertions
    ├── Replanning
    ├── Trajectory Storage
    └── Evaluation

Planner → Playwright State Extraction → Decision Loop → Assertions/Goal Detection → Replanning → Trajectory Storage