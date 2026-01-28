# Titanic Build Plan (Step-by-Step Execution Guide)

This document expands the end-to-end work plan for **Titanic** and walks through each task in
order. Each section includes **components needed**, **implementation steps**, and **completion
criteria** so the work can be executed incrementally and verified. A final “execution log”
template is included to track progress as tasks are completed.

---

## 0) Project Framing & Success Criteria

**Objective:** Ensure the project is installable, cross-platform, and satisfies the full feature
set.

**Tasks**
- [ ] Define success criteria and deliverables.
- [ ] Confirm npm-based installation requirement.

**Implementation steps**
1. Write `README.md` with goals, feature list, and quickstart.
2. Define CLI commands (`titanic start`, `titanic status`, `titanic stop`).
3. Document OS + Node.js version requirements.

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
- [ ] Define monorepo structure:
  - `packages/core` (MCP bus, registry, policy, loop)
  - `packages/cli` (CLI entrypoint)
  - `packages/ui` (web/desktop UI)
  - `packages/memory` (RAG + episodic)
  - `packages/models` (model connectors + feasibility)
  - `packages/sandbox` (filesystem + browser tools)
- [ ] Select runtime baseline: Node.js 18+ with TypeScript.
- [ ] Declare CLI entrypoint (`bin`) and start command.
- [ ] Plan CI workflow (build/test/publish).

**Implementation steps**
1. Create root `package.json` + `pnpm-workspace.yaml`.
2. Bootstrap each package with `package.json`, `tsconfig.json`.
3. Wire `packages/cli` to invoke `packages/core` start command.
4. Add `changesets` or `npm version` flow.

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
- [ ] Define `MCPMessage`, `ServiceDescriptor`, `PolicyDecision`, `TaskPlan`.
- [ ] Specify capability tagging schema for registry.
- [ ] Define transport-agnostic event bus interfaces.

**Implementation steps**
1. Create `packages/core/src/types.ts`.
2. Use `zod` or `ajv` for schema validation.
3. Document types in `/docs/contracts.md`.

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
- [ ] Implement message schema validation.
- [ ] Add routing strategies:
  - Service ID routing
  - Capability routing
  - Planner fallback
- [ ] Add tracing hooks and message logs.

**Implementation steps**
1. Add `McpBus` class with `publish`, `subscribe`, `route`.
2. Add `Router` with routing strategy chain.
3. Add `Logger` + `TraceContext`.

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
- [ ] Implement registry CRUD.
- [ ] Store metadata (cost, latency, sandbox, auth requirements).
- [ ] Add heartbeats and liveness checks.

**Implementation steps**
1. `ServiceRegistry` in memory with `register`, `deregister`, `list`, `lookup`.
2. Add TTL-based health checks.
3. Add persistence layer (SQLite or JSON).

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
- [ ] Implement `lookup(capability | service_id | tag)`.
- [ ] Add caching for fast lookups.
- [ ] Integrate lookup with MCP bus routing.

**Implementation steps**
1. Implement `lookup` in registry with query filters.
2. Add LRU cache for results.
3. Connect MCP bus router to registry lookup.

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
- [ ] Define policy DSL for allow/deny/rate limit.
- [ ] Implement decision engine with audit logs.
- [ ] Support policy override workflow.

**Implementation steps**
1. Create `policies/` folder with example rules.
2. Add `PolicyEngine.evaluate(action, context)`.
3. Store decision logs in `logs/policy.log`.

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
- [ ] Implement connectors for APIs and local models (HF + GGUF).
- [ ] Track model metadata: context length, cost/token, RAM/GPU requirements.
- [ ] Build feasibility engine that ranks models per constraints.
- [ ] Auto-download best model if missing.

**Implementation steps**
1. Define `ModelProvider` interface (OpenAI, Anthropic, HF, Local).
2. Add `ModelCatalog` with capability metadata.
3. Implement `selectBestModel(constraints)`.
4. Add `ModelDownloader` with caching.

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
- [ ] Define UI stack (React + Vite or desktop shell).
- [ ] Add task dashboard, memory explorer, policy view, registry status.
- [ ] Allow model/budget configuration and loop control.

**Implementation steps**
1. Create `packages/ui` with React + Vite.
2. Add pages: Dashboard, Memory, Policies, Registry.
3. Wire UI to core via REST/WebSocket API.

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
- [ ] Implement task decomposition and plan generation.
- [ ] Execute plan steps with tool integrations.
- [ ] Verify outputs and update status.
- [ ] Enforce cost/time/iteration caps.

**Implementation steps**
1. Implement `Planner.plan(goal)` -> `TaskPlan`.
2. Implement `Executor.run(plan)`.
3. Implement `Verifier.check(stepResult)`.
4. Add `LoopController` with guardrails.

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
- [ ] Implement local vector store and embedding pipelines.
- [ ] Create episodic session summaries.
- [ ] Add retrieval for planner and UI.

**Implementation steps**
1. Add embedding pipeline (OpenAI/HF).
2. Use SQLite + vectordb for storage.
3. Store summaries in `memory/episodes.json`.

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
- [ ] Define constitution file for behavioral rules.
- [ ] Add loop tripwires (max iterations, repeated failure).
- [ ] Provide safe fallback actions (pause, request approval).

**Implementation steps**
1. Create `config/constitution.yaml`.
2. Add `Tripwire` checks in loop controller.
3. Implement pause/escalation strategies.

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
- [ ] Implement restricted file read/write/exec sandbox.
- [ ] Allow package installation with limits.
- [ ] Integrate Playwright/Puppeteer for browser automation.
- [ ] Provide tool APIs (`readFile`, `writeFile`, `exec`, `browse`, `install`).

**Implementation steps**
1. Create `Sandbox` API with allowlisted directories.
2. Add command runner with timeouts.
3. Integrate Playwright with a safe wrapper.

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
- [ ] Boot core daemon, registry, UI from CLI.
- [ ] Ensure MCP bus orchestrates all service calls.
- [ ] Connect memory and policy to execution loop.

**Implementation steps**
1. Add `titanic start` to boot core + UI.
2. Ensure policy engine intercepts all tool calls.
3. Add startup health checks for registry + memory.

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
- [ ] Implement unit tests for MCP bus/registry/policy.
- [ ] Add integration tests for memory and loop.
- [ ] Add cross-platform CI matrix (Win/macOS/Linux).

**Implementation steps**
1. Add `vitest` for unit tests.
2. Use `playwright` for UI E2E tests.
3. Add GitHub Actions workflow for Windows/macOS/Linux.

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
- [ ] Write README and quickstart guides.
- [ ] Provide docs for service authoring & policies.
- [ ] Add contributing guide and code of conduct.

**Implementation steps**
1. Add `/docs/quickstart.md`.
2. Add `/docs/services.md` + `/docs/policies.md`.
3. Add `CONTRIBUTING.md` + `CODE_OF_CONDUCT.md`.

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

---

## Execution Log Template

Use this section to mark progress as work is completed.

| Task Area | Status | Notes | Owner | Date |
| --- | --- | --- | --- | --- |
| Repo foundations | ☐ Not started |  |  |  |
| Core architecture | ☐ Not started |  |  |  |
| MCP bus | ☐ Not started |  |  |  |
| Service registry | ☐ Not started |  |  |  |
| MCP lookup | ☐ Not started |  |  |  |
| Policy engine | ☐ Not started |  |  |  |
| Model feasibility | ☐ Not started |  |  |  |
| UI | ☐ Not started |  |  |  |
| Execution loop | ☐ Not started |  |  |  |
| Memory | ☐ Not started |  |  |  |
| Constitution & guardrails | ☐ Not started |  |  |  |
| Sandbox & browser | ☐ Not started |  |  |  |
| Integration | ☐ Not started |  |  |  |
| Testing | ☐ Not started |  |  |  |
| Documentation | ☐ Not started |  |  |  |
