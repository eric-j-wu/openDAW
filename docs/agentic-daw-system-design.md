# Agentic Cloud DAW — System Design Blueprint

## 1) Product strategy

### Vision
Build a production-grade, browser-first DAW with strong local performance, collaborative editing, cloud persistence, and an automation surface that can later be safely controlled by backend AI agents.

### Core principles
1. Audio never blocks on network.
2. Deterministic project state and history.
3. Explicit boundaries between UI, editing model, real-time engine, and cloud services.
4. AI hooks are event-driven, permissioned, and auditable.

## 2) Architecture overview

Adopt a layered architecture inspired by openDAW’s package split:

- **UI App Layer**: timeline, mixer, editors, dialogs, settings, scripting IDE.
- **Domain Layer**: project model, editing commands, undo/redo, validations, migrations.
- **Realtime Audio Layer**: AudioContext graph + AudioWorklet processors for render-critical DSP.
- **Background Worker Layer**: OPFS I/O, peak generation, transient detection, file processing.
- **Persistence Layer**: OPFS local storage, bundle import/export, optional cloud backup.
- **Collaboration Layer**: CRDT sync service + signaling.
- **Integration/API Layer**: internal command/event bus + external API gateway for future agentic backend.
- **Observability/Safety Layer**: metrics, crash recovery, access control, policy engine.

## 3) Technology choices

### Frontend runtime
- **TypeScript** end-to-end.
- **Vite** for app builds and fast dev loop.
- Lightweight custom UI primitives (optionally React/Solid if hiring pipeline demands it).
- **Sass/CSS modules** for controllable theming.

### Realtime audio
- **Web Audio API + AudioWorklet** for low-latency playback/processing.
- Keep critical DSP in worklets only.
- SharedArrayBuffer/ring-buffer channel for high-throughput worklet communication.

### Background processing
- **Web Workers** for non-realtime CPU tasks (waveform peaks, import transforms, file processing).
- Dedicated worker protocol contract, versioned.

### State and model
- Immutable/transactional graph model for project entities.
- Command-sourced editing surface (commands become API hooks later).

### Storage and sync
- **OPFS** as primary local persistence.
- Bundle format for import/export and backup.
- **Yjs + y-websocket** for collaboration.

### Backend (for agentic extension)
- **API gateway** (Node/TypeScript initially) with auth, rate limits, policy checks.
- **Event stream** (NATS/Kafka/Redis Streams) for asynchronous agent workflows.
- **Postgres** for metadata and audit logs.
- **Object storage** (S3-compatible) for assets/renders.

## 4) Domain model

Define stable core entities:
- Project
- Timeline
- AudioUnit (instrument/effect bus)
- Track (note/audio/value)
- Region/Clip/Event
- Devices and parameters
- Assets (samples, soundfonts, stems)
- User/session/collaboration state

Use:
- UUID IDs everywhere.
- Explicit schema versions.
- Migration pipeline for forward compatibility.

## 5) Engine design

### Real-time constraints
- No allocation-heavy logic inside render quantum.
- No async/network inside audio thread.
- Avoid locks; prefer lock-free queues/ring buffers.

### Components
- **EngineFacade** on UI thread controls play/stop/position and subscriptions.
- **EngineWorklet** holds transport, scheduling, processor graph.
- **Device processors** implement instrument/effect DSP with typed params.
- **Meter/recording worklets** isolated from main engine.

### Offline rendering
- Separate offline engine path for bounce/stems/export to keep UX responsive and deterministic.

## 6) Data, persistence, and recovery

### Local-first persistence
- Auto-save snapshots and commit log.
- OPFS path convention by project UUID.
- Periodic compaction (snapshot + delta truncation).

### Recovery
- Crash-safe write protocol (temp + atomic rename semantics where available).
- Startup recovery flow that can restore latest consistent state.

### Backup and cloud
- Incremental upload of project and assets.
- Separate metadata vs blob sync.
- Conflict policy: CRDT for collaborative model, explicit resolution for binary assets.

## 7) Collaboration architecture

- CRDT document stores editable project graph projection.
- Presence/awareness channel for cursors and selections.
- Role model: owner/editor/viewer.
- Room lifecycle: create/join/leave/archive.
- Security: signed room tokens, per-room ACL.

## 8) API and AI-hook architecture

Design AI integration now, even if enabled later.

### Internal API first
Build a typed internal command/event bus:
- Commands: `create_track`, `insert_device`, `set_parameter`, `split_region`, `export_stems`, etc.
- Events: `playback_started`, `clip_added`, `mixdown_completed`, `render_failed`, etc.

### External agent gateway
Expose agent-safe endpoints:
- `/v1/projects/:id/commands` (validated command envelope)
- `/v1/projects/:id/events` (webhook/stream)
- `/v1/projects/:id/artifacts` (renders/stems/analysis)

### Guardrails
- Permissioned command scopes.
- Human-in-the-loop mode for destructive commands.
- Idempotency keys.
- Full audit log (who/what/when/why).
- Policy engine for rate/content/scope restrictions.

## 9) Security and trust

- OAuth2/OIDC user auth.
- Workspace/project RBAC.
- Encryption in transit + at rest.
- Signed URLs for asset download/upload.
- Secret isolation for provider integrations.
- Action provenance and non-repudiation for agent operations.

## 10) Observability and SRE

- Structured logs (frontend + backend correlation IDs).
- Metrics: load time, XRuns/dropouts proxy, render durations, sync latency.
- Tracing for command lifecycle.
- Error budgets and SLOs:
  - open project latency
  - collaboration convergence time
  - export success rate

## 11) Recommended repo/workspace layout

- `apps/studio-web` (main DAW UI)
- `packages/audio-engine` (engine facade + worklet contracts)
- `packages/audio-processors` (DSP processors)
- `packages/workers` (OPFS/peaks/transients workers)
- `packages/domain-model` (project schema/entities/migrations)
- `packages/collab` (CRDT mapping + providers)
- `packages/sdk` (public plugin + automation SDK)
- `services/api-gateway` (auth, command ingress)
- `services/agent-orchestrator` (background automation jobs)
- `services/collab-server` (websocket/yjs)

## 12) Implementation phases

### Phase 1 — DAW foundation
- Transport, timeline, track/device graph, editing, undo/redo.
- Local-only persistence, import/export, offline bounce.
- Baseline effects/instruments and MIDI/audio recording.

### Phase 2 — Cloud and collaboration
- Accounts/workspaces/projects.
- Cloud backup, project browser, sharing.
- Realtime collaborative editing and presence.

### Phase 3 — API and automation
- Internal command/event bus formalized.
- Public API gateway with auth/policies.
- Webhook/event subscriptions and artifact APIs.

### Phase 4 — Agentic workflows
- Agent executor with scoped capabilities.
- Approval workflows and dry-run plans.
- Agent templates: “clean session”, “gain staging”, “stem prep”, “arrangement draft”.

## 13) Non-functional targets (initial)

- First-sound under 2 seconds on modern desktop.
- Stable operation at 128–256 buffer for common projects.
- No data loss across tab crashes with auto-save enabled.
- Deterministic export parity between online/offline paths.

## 14) Key design decisions to lock early

1. Stable project schema + migration story.
2. Command/event abstraction as the single editing gateway.
3. Worklet/worker protocol contracts versioned from day one.
4. Local-first persistence before cloud coupling.
5. Security/audit model before enabling external agents.

## 15) Why this fits openDAW as reference

openDAW already demonstrates important patterns worth retaining:
- Monorepo package boundaries for core/runtime/processors/workers.
- Browser capability gating for advanced APIs (AudioWorklet, OPFS, SharedArrayBuffer).
- Separate worker and audio worklet installation model.
- CRDT-based collaboration path and dedicated websocket service.

This blueprint keeps those strengths, but adds explicit API, policy, and orchestration layers so your future backend agent can integrate safely without compromising realtime UX or project integrity.
