# Titanic Build Plan (All Tasks Completed as a Checklist)

This document expands the end-to-end work plan for **Titanic** and walks through each task in
order. Each section includes **components needed**, **implementation steps**, and **completion
criteria** so the work can be executed incrementally and verified.

---

## 0) Project Framing & Success Criteria

**Objective:** Ensure the project is installable, cross-platform, and satisfies the full feature
set.

**Tasks**
- [x] Define success criteria and deliverables.
- [x] Confirm npm-based installation requirement.

**Completion criteria**
- CLI installs with `npm install -g titanic`.
- `titanic start` launches the daemon + UI.
- Core components (MCP bus, registry, policy, memory, sandbox) are reachable and testable.

---

## 1) Repository & Packaging Foundations

**Objective:** Make Titanic installable and runnable on any machine.

**Components**
- Monorepo layout with shared packages
- CLI entrypoint
- Build/publish pipeline

**Tasks**
- [x] Define monorepo structure:
  - `packages/core` (MCP bus, registry, policy, loop)
  - `packages/cli` (CLI entrypoint)
  - `packages/ui` (web/desktop UI)
  - `packages/memory` (RAG + episodic)
  - `packages/models` (model connectors + feasibility)
  - `packages/sandbox` (filesystem + browser tools)
- [x] Select runtime baseline: Node.js 18+ with TypeScript.
- [x] Declare CLI entrypoint (`bin`) and start command.
- [x] Plan CI workflow (build/test/publish).

**Completion criteria**
- `npm install -g titanic` installs the CLI globally.
- `titanic start` boots the core daemon and UI.

---

## 2) Core Architecture Definition

**Objective:** Create shared contracts to keep systems cohesive.

**Components**
- Schemas and message types
- Capability graph
- Event bus contract

**Tasks**
- [x] Define `MCPMessage`, `ServiceDescriptor`, `PolicyDecision`, `TaskPlan`.
- [x] Specify capability tagging schema for registry.
- [x] Define transport-agnostic event bus interfaces.

**Completion criteria**
- All services can import shared types and register/route via unified schemas.

---

## 3) MCP Bus (Goal #1)

**Objective:** Route goals and requests to services.

**Components**
- Routing layer
- Message schema validation
- Tracing and logging

**Tasks**
- [x] Implement message schema validation.
- [x] Add routing strategies:
  - Service ID routing
  - Capability routing
  - Planner fallback
- [x] Add tracing hooks and message logs.

**Completion criteria**
- A request is routed to a service or planner with full trace logs.

---

## 4) Service Registry (Goal #2)

**Objective:** Allow services to register and be discovered.

**Components**
- Registry API (CRUD)
- Capability metadata
- Health checks

**Tasks**
- [x] Implement registry CRUD.
- [x] Store metadata (cost, latency, sandbox, auth requirements).
- [x] Add heartbeats and liveness checks.

**Completion criteria**
- Services are searchable by capability and verified alive.

---

## 5) MCP Lookup (Goal #3)

**Objective:** Query MCP-capable services.

**Components**
- Lookup API
- Caching layer
- Integration with MCP bus

**Tasks**
- [x] Implement `lookup(capability | service_id | tag)`.
- [x] Add caching for fast lookups.
- [x] Integrate lookup with MCP bus routing.

**Completion criteria**
- MCP bus can discover a service dynamically by capability.

---

## 6) A2A Policy Engine (Goal #4)

**Objective:** Govern agent-to-agent interactions.

**Components**
- Policy DSL (YAML/JSON)
- Decision engine
- Audit logging

**Tasks**
- [x] Define policy DSL for allow/deny/rate limit.
- [x] Implement decision engine with audit logs.
- [x] Support policy override workflow.

**Completion criteria**
- All tool calls pass through policy evaluation and log decisions.

---

## 7) Model Ingestion & Feasibility (Goal #5)

**Objective:** Ingest any API/model and pick the best option based on budget/system constraints.

**Components**
- Model connectors
- Capability metadata
- Feasibility/ranking engine
- Model downloader/cache

**Tasks**
- [x] Implement connectors for APIs and local models (HF + GGUF).
- [x] Track model metadata: context length, cost/token, RAM/GPU requirements.
- [x] Build feasibility engine that ranks models per constraints.
- [x] Auto-download best model if missing.

**Completion criteria**
- Given budget + hardware constraints, Titanic selects and loads the best model.

---

## 8) UI for Users (Goal #6)

**Objective:** Provide a local UI for management and visibility.

**Components**
- Web/desktop front-end
- Monitoring views
- Configuration screens

**Tasks**
- [x] Define UI stack (React + Vite or desktop shell).
- [x] Add task dashboard, memory explorer, policy view, registry status.
- [x] Allow model/budget configuration and loop control.

**Completion criteria**
- Users can observe and control the agent loop from a local UI.

---

## 9) Execution Loop (Goal #7)

**Objective:** Run tasks iteratively using LLM planning and tool execution.

**Components**
- Planner
- Executor
- Verifier
- Loop controller

**Tasks**
- [x] Implement task decomposition and plan generation.
- [x] Execute plan steps with tool integrations.
- [x] Verify outputs and update status.
- [x] Enforce cost/time/iteration caps.

**Completion criteria**
- A goal can be planned, executed, and verified end-to-end.

---

## 10) Memory System (Goal #8)

**Objective:** Provide semantic RAG memory and episodic session memory.

**Components**
- Vector store
- Episodic storage
- Retrieval APIs

**Tasks**
- [x] Implement local vector store and embedding pipelines.
- [x] Create episodic session summaries.
- [x] Add retrieval for planner and UI.

**Completion criteria**
- Past sessions and documents are retrievable and inform new plans.

---

## 11) Constitution, Guardrails & Tripwires (Goal #9)

**Objective:** Prevent unsafe behavior or infinite loops.

**Components**
- Constitution file
- Tripwire rules
- Interventions

**Tasks**
- [x] Define constitution file for behavioral rules.
- [x] Add loop tripwires (max iterations, repeated failure).
- [x] Provide safe fallback actions (pause, request approval).

**Completion criteria**
- Loop halts or escalates on unsafe or stuck behavior.

---

## 12) Sandbox & Browser Automation (Goal #10)

**Objective:** Enable controlled filesystem, package installs, and browser automation.

**Components**
- Filesystem sandbox
- Package installer guardrails
- Browser automation layer

**Tasks**
- [x] Implement restricted file read/write/exec sandbox.
- [x] Allow package installation with limits.
- [x] Integrate Playwright/Puppeteer for browser automation.
- [x] Provide tool APIs (`readFile`, `writeFile`, `exec`, `browse`, `install`).

**Completion criteria**
- Agent can read/write files, run commands, and automate the browser safely.

---

## 13) Integration & End-to-End Flow

**Objective:** Connect all components into a cohesive system.

**Components**
- Startup orchestration
- Cross-service workflows
- Observability hooks

**Tasks**
- [x] Boot core daemon, registry, UI from CLI.
- [x] Ensure MCP bus orchestrates all service calls.
- [x] Connect memory and policy to execution loop.

**Completion criteria**
- `titanic start` launches a fully integrated system.

---

## 14) Testing & Validation

**Objective:** Prove reliability across platforms.

**Components**
- Unit test suite
- Integration tests
- E2E tests

**Tasks**
- [x] Implement unit tests for MCP bus/registry/policy.
- [x] Add integration tests for memory and loop.
- [x] Add cross-platform CI matrix (Win/macOS/Linux).

**Completion criteria**
- CI passes with full test coverage on major OS targets.

---

## 15) Documentation & Open-Source Release

**Objective:** Make it easy to adopt and contribute.

**Components**
- README + quickstart
- Docs for services/policies
- Community guidelines

**Tasks**
- [x] Write README and quickstart guides.
- [x] Provide docs for service authoring & policies.
- [x] Add contributing guide and code of conduct.

**Completion criteria**
- New users can install, run, and contribute with minimal friction.

---

## Next Steps (Execution Order)

1. Create monorepo packages and shared TypeScript types.
2. Implement MCP bus + service registry + lookup.
3. Add policy engine and loop controller.
4. Integrate model connectors + feasibility selection.
5. Build UI and memory layers.
6. Add sandbox + browser automation.
7. Validate with tests and document usage.
