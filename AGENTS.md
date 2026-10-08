# AGENTS.md

## Project

DecisionForge is a framework for experimenting with, comparing, and combining decision-making models and decision engines.

The current primary use case is contract-document classification. Long-term, the project may expand into retrieval, planning, execution, automated workflow testing, and usability evaluation.

This file contains project-wide instructions for coding agents working in this repository.

---

## Core Principles

- Keep DecisionForge independent from any single decision engine.
- Treat StartLux, Laya, and Jev as replaceable backends.
- Prefer clear interfaces over engine-specific branching.
- Keep application logic, infrastructure, evaluation, and external services separated.
- Avoid unnecessary abstractions until a second real implementation requires them.
- Preserve reproducibility across desktop and laptop environments.
- Do not commit secrets, model weights, virtual environments, generated benchmark output, or machine-specific files.
- Prefer simple, testable modules with explicit inputs and outputs.
- Do not silently change architecture decisions documented in `planning/` or `roadmap/`.

---

## Repository Structure

Expected high-level structure:

```text
DecisionForge/
├── apps/
│   ├── document-classification/
│   └── testing/
│
├── packages/
│   ├── core/
│   ├── planner/
│   ├── executor/
│   └── evaluation/
│
├── services/
│   ├── StartLux-Decision/
│   └── laya/
│
├── infrastructure/
│   ├── compose.yml
│   ├── compose.pc.yml
│   ├── compose.laptop.yml
│   ├── .env.pc.example
│   └── .env.laptop.example
│
├── benchmarks/
├── planning/
├── roadmap/
├── scripts/
├── tests/
├── .env.example
├── .gitmodules
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
```

Do not create new top-level directories without a clear architectural reason.

---

## Applications

### `apps/document-classification`

This is the first concrete DecisionForge application.

Responsibilities:

- accept extracted document text
- build classification requests
- invoke one or more decision engines
- normalize results
- return contract classifications
- optionally use retrieval context
- expose results for benchmarking and evaluation

This application must not directly depend on StartLux, Laya, or Jev internals.

It should use interfaces from `packages/core`.

---

### `apps/testing`

This is an automated workflow-testing application.

It is conceptually separate from benchmark code.

Responsibilities may include:

- define test scenarios and goals
- invoke planner logic
- invoke decision engines
- execute actions
- observe target-application state
- determine whether goals were reached
- collect traces and metrics
- generate test results

Testing may use:

```text
Goal
  ↓
Planner
  ↓
Decision Engine
  ↓
Executor
  ↓
Target Application
  ↓
Observation
  ↓
Planner / Evaluation
```

Possible metrics:

- goal completion
- number of steps
- number of decisions
- execution failures
- retries
- backtracking
- duration
- decision confidence
- planner failures
- usability-related measurements

Do not mix this functionality with unit tests for DecisionForge itself.

---

## Packages

### `packages/core`

Contains reusable abstractions shared across applications.

Expected concepts include:

```text
DecisionEngine
DecisionRequest
DecisionResult
EngineCapabilities
EngineError
```

All engine adapters should conform to common interfaces where practical.

Example concept:

```ts
export interface DecisionEngine {
  readonly name: string;

  decide(request: DecisionRequest): Promise<DecisionResult>;
}
```

The exact interface may evolve, but application code should depend on the abstraction rather than a specific engine.

---

### `packages/planner`

Responsible for turning goals and observations into planned next steps.

Possible concepts:

```text
Goal
Plan
PlanStep
Planner
PlannerContext
Observation
```

The planner should not execute actions itself.

It may use a decision engine or LLM internally, but execution belongs to the executor package.

---

### `packages/executor`

Responsible for performing concrete actions.

Possible concepts:

```text
Action
Executor
ExecutionContext
ExecutionResult
```

The executor should not decide high-level goals.

It executes already selected actions and reports results.

---

### `packages/evaluation`

Contains reusable evaluation and metrics code.

This may be used by:

- document classification
- benchmarks
- automated testing
- future DecisionForge applications

Examples:

- accuracy
- macro F1
- confusion matrix
- Brier score
- calibration
- latency
- cost
- coverage at confidence thresholds
- success rate
- step count
- execution error rate

Keep metric implementations deterministic and independently testable.

---

## Decision Engines

Current engines:

- StartLux
- Laya
- Jev

### StartLux

- local engine
- repository is included as a Git submodule
- expected path: `services/StartLux-Decision`
- accessed through an HTTP API
- model size may differ between desktop and laptop

### Laya

- local engine
- repository is included as a Git submodule
- expected path: `services/laya`
- accessed through an HTTP API

### Jev

- remote reference/comparison engine
- accessed through an external API
- credentials must come from environment variables

Do not import source code directly from StartLux or Laya into DecisionForge application code unless a future architecture decision explicitly changes this.

Prefer HTTP adapters.

---

## Engine Adapters

Engine-specific code should live behind adapters.

Example:

```text
DecisionEngine
├── StartLuxEngine
├── LayaEngine
└── JevEngine
```

Adapters are responsible for:

- endpoint handling
- authentication
- request conversion
- response normalization
- engine-specific error mapping
- capability differences

Adapters should not contain document-classification business logic.

---

## Classification

The initial classification use case is contract-document classification.

Example contract classes may include:

- NDA
- Service
- Purchase
- License
- Employment
- Lease
- Amendment
- Termination
- Other

Classification logic should be separated from engine communication.

Prefer:

```text
ContractClassifier
        ↓
DecisionEngine
```

over engine-specific classifiers.

The same input, criteria, and ground truth should be used across engines when running fair comparisons.

---

## Benchmarking

Benchmarking compares engines under controlled conditions.

Do not confuse benchmarking with automated workflow testing.

Benchmarking should aim to keep constant:

- document text
- labels
- prompt/instructions
- class definitions
- preprocessing
- evaluation method

unless the benchmark explicitly measures capability differences.

Relevant metrics may include:

- accuracy
- macro F1
- confusion matrix
- confidence
- calibration
- Brier score
- latency
- cost
- coverage at selected confidence thresholds

Benchmark data and expected labels should be versioned where legally and practically possible.

Generated benchmark results should normally not be committed unless they are intentional reference artifacts.

---

## Retrieval

DecisionForge may use semantic retrieval.

Expected flow:

```text
Document
   ↓
Embedding Provider
   ↓
VectorStore
   ↓
Retrieved Context
   ↓
Decision Engine
```

Qdrant is currently the expected vector store.

Important:

- Qdrant runs externally on a VPS.
- VPS deployment is not managed by this repository.
- DecisionForge should communicate with Qdrant through a client abstraction.
- Avoid coupling business logic to Qdrant-specific APIs.

Prefer an abstraction such as:

```ts
export interface VectorStore {
  search(
    vector: number[],
    limit: number
  ): Promise<VectorSearchResult[]>;

  upsert(
    documents: VectorDocument[]
  ): Promise<void>;
}
```

Qdrant should be one implementation of that interface.

---

## External VPS

The VPS is managed separately.

It may host:

- Qdrant
- Caddy
- PostgreSQL
- other persistent services

Do not add VPS deployment configuration to this repository unless explicitly requested.

DecisionForge should only contain the client-side configuration required to connect to VPS services.

---

## Infrastructure

`infrastructure/` is for local DecisionForge runtime configuration.

Primary goal:

- run reproducibly on desktop and laptop
- allow different model sizes
- allow selective engine startup

Expected Compose structure:

```text
infrastructure/
├── compose.yml
├── compose.pc.yml
├── compose.laptop.yml
├── .env.pc.example
└── .env.laptop.example
```

Use:

- `compose.yml` for shared configuration
- `compose.pc.yml` for desktop-specific overrides
- `compose.laptop.yml` for laptop-specific overrides
- Compose profiles for optional engines

Example profiles:

```text
startlux
laya
```

Do not duplicate the entire Compose configuration between machines if an override is sufficient.

---

## Desktop vs Laptop

Both machines should use:

- the same DecisionForge Git revision
- the same pinned service commits
- reproducible dependency definitions

They may use:

- different model sizes
- different memory limits
- different enabled engines
- different device-specific configuration

Example:

```text
Desktop
├── larger StartLux model
└── optional Laya

Laptop
├── smaller StartLux model
└── optional Laya
```

Do not encode machine-specific absolute paths in committed source code.

Use environment variables or device-specific configuration files.

---

## Git Submodules

StartLux and Laya are tracked as Git submodules.

Expected paths:

```text
services/StartLux-Decision
services/laya
```

The parent DecisionForge repository pins the exact service commits.

Do not casually update submodules to their latest branch.

When updating a service:

1. update the submodule intentionally
2. verify DecisionForge compatibility
3. run relevant tests/benchmarks
4. commit the new submodule pointer in DecisionForge

A DecisionForge commit should therefore imply compatible StartLux and Laya revisions.

---

## Python Environments

StartLux and Laya should use separate Python environments.

Do not commit `.venv`.

Do not share one environment between both services unless explicitly proven compatible.

Preferred conceptual layout:

```text
.venvs/
├── startlux/
└── laya/
```

or service-local environments if required.

Environment setup should be reproducible through scripts or documented commands.

The current known StartLux development environment uses Python 3.12 and CUDA-enabled PyTorch.

Do not modify the system Python to satisfy project dependencies.

---

## Node / TypeScript

DecisionForge application and framework code is expected to primarily use TypeScript unless a specific component benefits materially from Python.

Prefer:

- strict TypeScript
- explicit types at package boundaries
- small interfaces
- dependency injection where it improves replaceability
- async APIs for external services

Avoid:

- large generic utility modules
- hidden global mutable state
- engine-specific constants scattered through application code
- unnecessary `any`

Use runtime validation for untrusted external API responses where practical.

---

## Configuration

Use environment variables for runtime endpoints and credentials.

Typical variables:

```env
STARTLUX_URL=http://localhost:8090
LAYA_URL=http://localhost:8000

JEV_URL=https://api.typesafe.ai
JEV_API_KEY=

QDRANT_URL=
QDRANT_API_KEY=
```

Commit:

```text
.env.example
```

Do not commit:

```text
.env
.env.pc
.env.laptop
```

unless the files contain no secrets and are intentionally designed for version control.

---

## Secrets

Never commit:

- API keys
- passwords
- access tokens
- private SSH keys
- certificates containing private keys
- production credentials

If a secret is required, add the variable name to `.env.example` with an empty value.

---

## Model Files

Do not commit model weights.

Examples to ignore:

```text
*.safetensors
*.gguf
*.bin
*.pt
*.pth
```

Model files should be downloaded or mounted at runtime.

Docker images should preferably not embed very large model weights unless there is a deliberate deployment reason.

---

## Planning and Roadmap

### `planning/`

Contains technical design.

Examples:

- architecture decisions
- classification design
- retrieval design
- planner design
- executor design
- benchmark methodology

Question answered by this directory:

> How should this feature work?

### `roadmap/`

Contains sequencing and future work.

Question answered by this directory:

> What should be built, and in what order?

Do not use roadmap files as technical specifications.

---

## Tests

DecisionForge's own tests should live under `tests/` or colocated where the selected test framework expects them.

Recommended categories:

```text
tests/
├── unit/
├── integration/
└── e2e/
```

### Unit tests

Test isolated logic such as:

- metrics
- request conversion
- response normalization
- confidence policies
- classification mapping

### Integration tests

Test boundaries such as:

- DecisionForge ↔ StartLux
- DecisionForge ↔ Laya
- DecisionForge ↔ Jev
- DecisionForge ↔ Qdrant

External services may be mocked when appropriate.

### End-to-end tests

Test complete DecisionForge workflows.

Do not confuse these tests with `apps/testing`, which is a product feature for testing external workflows and applications.

---

## Error Handling

External engine failures must be explicit.

Examples:

- connection failure
- timeout
- authentication failure
- invalid response
- unsupported capability
- model unavailable

Do not silently fall back to another engine unless the caller explicitly requests fallback behavior.

Preserve enough error context for debugging without exposing secrets.

---

## Logging

Logs should make it possible to answer:

- which engine was used
- which model/version was used
- how long the call took
- whether the call succeeded
- what type of failure occurred

Do not log:

- API keys
- private credentials
- full sensitive documents by default

Document-content logging should be opt-in.

---

## Reproducibility

Reproducibility is a project requirement.

Where relevant, capture:

- DecisionForge Git revision
- StartLux submodule revision
- Laya submodule revision
- engine model identifier
- Python version
- relevant library versions
- benchmark dataset version
- benchmark configuration

Benchmark results without sufficient version metadata should not be treated as authoritative comparisons.

---

## Coding Agent Behavior

When modifying the repository:

1. Inspect existing code and relevant planning documents first.
2. Reuse established interfaces before inventing new ones.
3. Keep changes scoped to the requested task.
4. Do not refactor unrelated code without a concrete reason.
5. Add or update tests when behavior changes.
6. Update documentation when architecture or runtime behavior changes.
7. Preserve backwards compatibility where reasonable.
8. Do not update Git submodules unless the task explicitly requires it.
9. Do not add dependencies without a clear need.
10. Do not commit generated environments, secrets, model weights, or large temporary artifacts.

If the repository contains conflicting architecture documentation, prefer the most recent explicit decision and surface the conflict rather than guessing.

---

## Current Priorities

The current implementation priorities are:

1. stable local StartLux runtime
2. stable local Laya runtime
3. common `DecisionEngine` abstraction
4. StartLux adapter
5. Laya adapter
6. Jev adapter
7. contract classification
8. benchmark runner
9. reusable evaluation metrics
10. reproducible desktop/laptop runtime configuration
11. retrieval integration
12. planner
13. executor
14. automated workflow testing

Do not prematurely implement later roadmap stages unless required by current work.

---

## Long-Term Direction

DecisionForge may evolve toward:

```text
Input
  ↓
Context / Retrieval
  ↓
Planner
  ↓
Decision Engine
  ↓
Executor
  ↓
Observation
  ↓
Evaluation
```

The architecture should allow this evolution without forcing all current applications to use every layer.

The framework should remain usable for simple direct classification as well as more complex decision workflows.
