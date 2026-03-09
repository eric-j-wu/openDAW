# Cloud DAW System Design Blueprint (Greenfield / Clean-Room)

This blueprint is for building a production-grade cloud DAW **from scratch**. Existing DAW projects are only reference input for high-level ideas, never a source for direct code, copied structures, or lifted assets.

## 1. Clean-room constraints (mandatory)

- Do not copy code, file layouts, identifiers, tests, UI assets, or docs from existing DAW repositories.
- Treat external projects as product research only (feature inventory, quality bars, UX patterns).
- Re-implement all systems with original architecture decisions and original code.
- Maintain a traceable decision log proving independent implementation.

## 2. Product architecture goals

Design for four independent but connected planes:

1. **Realtime Audio Plane (hard realtime)**
2. **Creative Interaction Plane (soft realtime UI)**
3. **Data & Collaboration Plane**
4. **Intelligence & Automation Plane**

Each plane should be independently deployable and testable.

## 3. Recommended technology choices (greenfield)

### 3.1 Frontend + DSP
- **Language:** TypeScript end-to-end.
- **Web stack:** Vite + modular UI architecture.
- **Audio:** WebAudio + AudioWorklet + Worker split.
- **Optional acceleration:** Rust/C++ WASM for hotspots only.

### 3.2 Backend platform
- **API implementation:** TypeScript (Fastify/Nest) or Go.
- **API style:** internal RPC + public REST + WebSocket events.
- **Collab core:** Yjs-compatible CRDT service with Redis-backed presence.
- **Async jobs:** NATS JetStream/Kafka class queue.
- **Storage:** PostgreSQL + S3-compatible objects + Redis.

### 3.3 AI integration
- JSON-schema versioned tool contracts.
- Async command execution + event subscriptions.
- Policy-gated write operations.

## 4. Macro architecture

## Clients
- Web DAW
- Companion control clients
- API automation clients

## Platform services
- Identity/Tenant
- Project
- Asset
- Render
- Collaboration
- Agent Orchestrator
- Audit/Event

## Data systems
- PostgreSQL
- Object storage
- Redis
- Queue
- Observability stack

## 5. Bounded contexts

- **Composition:** timeline, clips, arrangement, automation.
- **Sound:** devices, parameters, routing.
- **Media:** sample/media lifecycle, transcoding metadata, peaks.
- **Session:** users, presence, permissions.
- **Automation:** macro execution, agent actions, approvals.

## 6. Realtime architecture requirements

- Never block audio callback on network/storage/AI calls.
- Use deterministic command channel UI → engine.
- Emit bounded-rate state snapshots engine → UI.
- Preserve sample-accurate timing for transport/automation/MIDI.

Target SLOs:
- Audio graph startup < 300ms (modern desktop baseline).
- p95 UI→audio command latency < 40ms.
- Dropout rate < 1 in 10 minutes under nominal load.

## 7. Persistence + versioning

- Versioned canonical project schema.
- Snapshot + operation model.
- Immutable media references.
- Migration pipeline with fixture-based regression tests.

Save modes:
1. local autosave
2. cloud checkpoint
3. publish/export artifact

## 8. Collaboration design

- CRDT for shared editable state.
- Large binary media stays outside CRDT payloads.
- Separate channels for sync, presence, signaling, and audit.
- Optional soft-lock hints for high-conflict regions.

## 9. Agent API model

### Read tools
- `get_project_structure`
- `get_track_state`
- `get_selection`
- `get_mixer_snapshot`

### Write tools
- `create_track`
- `create_region`
- `insert_device`
- `set_parameter`
- `apply_macro`
- `render_preview`

All writes require dry-run support, idempotency, version checks, and audit emission.

## 10. Security + tenant isolation

- OAuth/OIDC auth model.
- Tenant-scoped RBAC/ABAC authorization.
- Signed URL model for object transfer.
- Immutable audit trail for user and agent actions.

## 11. Testing strategy

- Unit: domain logic and reducers.
- Contract: API/event/tool schemas.
- Audio regression: offline render fingerprinting.
- E2E: create/edit/save/render/collab critical paths.
- Chaos: websocket disconnects, queue delays, storage partial failures.

## 12. Build roadmap (high-level)

1. Core single-user engine/editor hardening.
2. Cloud persistence and asset ingestion.
3. Collaboration orchestration.
4. Agent read APIs then guarded write APIs.

## 13. Clean-room governance checklist

For each delivery milestone, record:
- architectural decisions made from first principles
- no-copy attestation for implementation
- contract/version changes
- risk log and mitigations
