# Unified Cloud DAW Master Plan (Greenfield + Agentic Execution)

This document defines a unified implementation-ready plan for building a cloud DAW from scratch, plus a **step-by-step execution sequence** for coding agents. It explicitly enforces clean-room implementation (no direct copying from existing DAW codebases).

## 0) Synthesis baseline

This unified plan is derived from:

- Your current architecture notes and product requirements.
- Industry-standard cloud DAW constraints (realtime, collaboration, media scale).
- Roadmap priorities for cloud features and AI-assisted workflows.

> If you have additional planning documents outside this repository, append them in a dedicated `docs/plans/` folder and extend this document’s “Decision Deltas” section after each review cycle.


## 0.1) Clean-room implementation policy (non-negotiable)

- Existing DAW repositories may be used for inspiration and benchmarking only.
- Do not copy code, file structures, identifiers, test fixtures, assets, or prose verbatim.
- All implementation artifacts must be original and traceable through ADRs and decision logs.
- Every agent task must include a “no-copy attestation” in its completion notes.

## 1) Product north star

Build a production-grade, web/cloud DAW that:

1. Preserves low-latency, deterministic realtime audio behavior in-browser.
2. Scales to cloud persistence, render pipelines, and collaboration.
3. Exposes safe, versioned APIs/hooks so backend agentic AI can observe and act on projects.
4. Maintains strict boundaries: **realtime audio plane never blocks on network or AI.**

## 2) Non-negotiable architecture principles

1. **Realtime isolation**
   - Audio worklet graph and scheduling are isolated from storage/network/AI variability.
2. **Contract-first external interfaces**
   - REST/OpenAPI, events/AsyncAPI, and AI tool contracts/JSON Schema are versioned artifacts.
3. **Domain separation**
   - Composition, Sound, Media, Session, and Automation domains evolve independently.
4. **Evented backend**
   - Mutations become auditable events and replayable command history where practical.
5. **Offline-first UX**
   - Local autosave and recovery always work even when cloud is unavailable.
6. **Safe AI write-path**
   - AI commands support dry-run, idempotency keys, approval policies, and rollback metadata.

## 3) Unified target architecture

## 3.1 Planes

### A. Realtime Audio Plane (hard realtime)
- Browser AudioContext + AudioWorklet processors + Worker infrastructure.
- Sample-accurate transport, automation, MIDI dispatch.
- Command ring-buffer from UI → engine, snapshot stream from engine → UI.

### B. Creative Interaction Plane (soft realtime)
- Timeline editors, mixer/devices, project editing commands.
- User-intent command layer (not direct mutation from arbitrary UI components).

### C. Data/Collaboration Plane
- Canonical project schema + migrations.
- Snapshot + incremental operations model.
- Cloud persistence and collaboration session state.

### D. Intelligence/Automation Plane
- Agent tools for read/write operations.
- Orchestrator for long-running tasks (render, analysis, batch edits).
- Policy gate for permissions and approvals.

## 3.2 Service map

- **API Gateway**: auth, tenant routing, rate limits.
- **Project Service**: metadata, checkpoints, project retrieval lifecycle.
- **Asset Service**: sample/media ingestion, object store refs, waveform metadata.
- **Collaboration Service**: Yjs room/session orchestration and awareness.
- **Render Service**: mixdown/stems/offline bounces.
- **Agent Orchestrator**: command execution flow, dry-run validation, policy checks.
- **Audit/Event Service**: immutable activity stream and compliance trail.

## 3.3 Data stack

- PostgreSQL: metadata, users/tenants, access control, audit index.
- Object store (S3-compatible): audio/media blobs and export artifacts.
- Redis: cache, presence, ephemeral collaboration/session state.
- Queue (NATS/Kafka class): async render/analysis jobs and retries.

## 4) Domain boundaries and ownership

## 4.1 Composition Domain
Owns:
- Tracks, regions/clips, arrangement timeline, markers, automation lanes.

Exposes:
- Pure command APIs (`createRegion`, `moveClip`, `splitRegion`, etc.).
- Deterministic reducers/validators.

## 4.2 Sound Domain
Owns:
- Device chains, parameter states, routing topology, presets binding.

Exposes:
- Graph-safe mutation commands.
- Parameter automation interface.

## 4.3 Media Domain
Owns:
- Sample identity, imports, transcoding outputs, peaks.

Exposes:
- Content-addressable references + metadata APIs.

## 4.4 Session Domain
Owns:
- User presence, roles/permissions, room participation, soft-locks.

Exposes:
- Collaboration session APIs and authz checks.

## 4.5 Automation Domain
Owns:
- Macro commands, AI command wrappers, approval workflows, execution traces.

Exposes:
- Tool contract adapters and policy-governed write pipelines.

## 5) Canonical project model strategy

Use a **hybrid snapshot + operation** model:

- Canonical schema version `projectSchemaVersion`.
- Optional operation log entries for replayable edits.
- Periodic compact snapshots for quick load.
- Immutable media references stored externally.

Required migration policy:
- Every schema bump ships with forward migration.
- CI runs migration fixtures from N-2 schema versions minimum.

## 6) AI/agent integration design

## 6.1 Read tools first
Phase-in read-only tools before write tools:

- `get_project_structure`
- `get_selection`
- `get_track_details`
- `get_device_chain`
- `get_mixer_snapshot`

## 6.2 Write tools (guarded)
All write tools must support:

- `dryRun: boolean`
- `idempotencyKey`
- `commandId`
- `expectedProjectVersion`
- `policyContext`
- `rollbackHint`

Starter write tool set:
- `create_track`
- `create_region`
- `insert_device`
- `set_parameter`
- `apply_macro`
- `render_preview`

## 6.3 Approval model
Policy tiers:
- **Tier 0:** read-only, always allowed.
- **Tier 1:** non-destructive writes, auto-approve with audit.
- **Tier 2:** destructive/large edits, explicit user approval required.

## 7) Unified implementation roadmap

## Phase 1 — Core hardening (single-user)
Goal: deterministic core + resilient local workflows.

- Formalize command bus between UI and engine.
- Separate UI mutation intents from domain mutations.
- Harden autosave/recovery semantics.
- Add golden-file migration tests.

Exit criteria:
- Stable load/save from local storage with migration coverage.
- Realtime path unaffected by cloud unavailability.

## Phase 2 — Cloud foundation
Goal: cloud project persistence and media lifecycle.

- Add Auth + tenant model.
- Add Project Service + Asset Service.
- Implement cloud checkpoints and resumable media upload.
- Add async render pipeline for mixdown/stems.

Exit criteria:
- Open/save cloud projects with consistent version checks.
- Render jobs recover from worker restarts.

## Phase 3 — Collaboration beta
Goal: real-time shared editing that remains usable under conflict.

- Deploy collaboration room orchestration.
- Presence and soft-lock hints on track/region edits.
- Activity feed via event stream.

Exit criteria:
- Two-user shared edit sessions with recoverable reconnect behavior.

## Phase 4 — Agent API alpha
Goal: safe automation with auditable AI actions.

- Publish read tool contracts and docs.
- Add guarded write tools + dry-run validations.
- Implement approval UX for high-impact actions.

Exit criteria:
- AI can perform bounded edits end-to-end with audit traceability.

## 8) Step-by-step agent handoff plan (sequential prompts)

Use these as linear work packages for coding agents. Each step has strict boundaries and expected outputs.

## Step 1: Clean-room scaffolding and boundary ADR
**Agent objective:** Establish original package boundaries, contracts skeletons, and clean-room documentation without behavior changes.

Deliver:
- `docs/adr/ADR-000-clean-room-policy.md` (implementation rules and attestation template).
- `packages/contracts/{openapi,asyncapi,tools}/` with starter schemas.
- `packages/domain/{composition,sound,media,session,automation}/` package stubs.
- ADR doc for boundaries and dependency rules.

Guardrails:
- No runtime behavior changes.
- No direct imports from app UI into domain packages.
- No copied code/text/asset fragments from external DAW repositories.

## Step 2: Command bus formalization
**Agent objective:** Introduce command envelope model between UI intent and core mutation.

Deliver:
- Command envelope types (`commandId`, timestamps, source, target domain).
- Validation layer + typed dispatch.
- Compatibility layer between initial UI intents and domain command envelopes.

Guardrails:
- Must preserve behavior validated by the baseline acceptance tests for this step.
- Must include unit tests for dispatch and validation failures.

## Step 3: Project schema versioning + migration harness
**Agent objective:** Build deterministic schema evolution workflow.

Deliver:
- Canonical schema version marker.
- Migration runner + fixtures for old schema snapshots.
- CI job for migration regression tests.

Guardrails:
- Migration must be pure + repeatable.
- No data loss on supported versions.

## Step 4: Cloud persistence MVP
**Agent objective:** Add project metadata API and object storage integration.

Deliver:
- Project Service endpoints: create/list/get checkpoint/update metadata.
- Asset upload/download flow using signed URLs.
- Client adapter for checkpoint sync.

Guardrails:
- Local mode must still work with cloud disabled.
- All APIs must be tenant-scoped.

## Step 5: Render service pipeline
**Agent objective:** Add queued async render jobs.

Deliver:
- Job enqueue API + worker execution for mixdown/stems.
- Status polling + completion events.
- Artifact metadata persistence.

Guardrails:
- Idempotent retries.
- Explicit timeout/retry/dead-letter strategy.

## Step 6: Collaboration orchestration
**Agent objective:** Productionize Yjs session orchestration.

Deliver:
- Room routing model (`tenantId/projectId/roomId`).
- Presence + heartbeat behavior.
- Conflict hints/soft-lock events.

Guardrails:
- Large media never embedded in CRDT state.
- Reconnect must resync correctly.

## Step 7: Audit/event backbone
**Agent objective:** Make all key mutations observable and traceable.

Deliver:
- Domain event publication for key project/media/collab actions.
- Audit storage with correlation IDs.
- Activity feed query endpoint.

Guardrails:
- Events versioned from first release.
- PII minimization in payloads.

## Step 8: Agent read APIs
**Agent objective:** Publish stable read-only tools for automation assistants.

Deliver:
- JSON schemas for read tools.
- Tool gateway implementation + authz.
- Example agent session docs.

Guardrails:
- Strict pagination/size limits.
- No implicit write side-effects.

## Step 9: Agent write APIs with policy gates
**Agent objective:** Enable safe, bounded write operations.

Deliver:
- Write tool contracts with dry-run + idempotency.
- Policy engine integration (tiered approvals).
- Rollback metadata and execution receipts.

Guardrails:
- Destructive commands require approval flows.
- All writes emit audit events.

## Step 10: Hardening and scale readiness
**Agent objective:** Make platform production-resilient.

Deliver:
- Load/perf tests for websocket fanout and render queue.
- Observability dashboards and SLO alerts.
- Security review checklist closure.

Guardrails:
- Realtime p95 budgets tracked per release.
- Rollback playbooks for each core service.

## 9) Prompt template for coding agents (copy/paste)

Use this exact structure for each step:

1. **Context**: “You are implementing Step X from `docs/cloud-daw-unified-master-plan.md`.”
2. **Scope**: list in-scope files/packages only.
3. **Out-of-scope**: explicit exclusions.
4. **Deliverables**: concrete artifacts + tests.
5. **Acceptance criteria**: measurable checks.
6. **Architecture guardrails**: domain boundaries + no forbidden dependencies.
7. **Validation commands**: required test/lint/build commands.
8. **PR checklist**: migration notes, API contract bumps, risk notes, no-copy attestation.

## 10) Governance loop (keep plan unified over time)

At the end of each completed step:

- Add a “Decision Delta” entry with:
  - What changed.
  - Which architecture principle it impacts.
  - Any contract/schema version bumps.
  - New risks + mitigations.
- Update this master plan only through additive deltas (avoid ad-hoc rewrites).

This keeps agent-generated implementation work aligned with a single source of truth.
