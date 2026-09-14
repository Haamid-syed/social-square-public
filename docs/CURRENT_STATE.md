# Current state and roadmap

This is an evidence-based capability matrix for the private implementation as verified on **14 September 2026**. It distinguishes working code from scaffolding and planned evolution.

## Status definitions

| Status | Meaning |
|---|---|
| **Implemented** | Connected to the current user/runtime path |
| **Partial** | Meaningful implementation exists, but a critical piece is disabled or missing |
| **Modeled** | Schema/types/UI concept exist, but not wired into live behavior |
| **Planned** | Architectural direction only; no complete current implementation |

## Capability matrix

| Capability | Status | Current evidence / boundary |
|---|---|---|
| Email/password signup and login | Implemented | Validation, hashing, user creation/lookup, cookies |
| Google and GitHub OAuth | Implemented | Start and callback routes with state validation |
| Access + refresh sessions | Implemented | 15-minute access, 7-day DB-backed refresh, logout revocation |
| Automatic access-token refresh UX | Partial | Refresh endpoint exists; full browser retry loop does not |
| Password reset | Partial | Secure token and reset path exist; transactional email does not |
| Student/remote-worker onboarding | Implemented | Profile branch and completion flag |
| Profile/settings/password change | Implemented | Authenticated handlers and UI |
| Avatar uploads | Partial | Validation/write/association work; storage is container-local and signup ownership path needs hardening |
| Protected routes | Implemented | Next.js proxy + Edge JWT verification + onboarding guard |
| Fresh production build | Blocked | Legacy middleware and current proxy coexist |
| Lint/type gates | Partial | Scripts/config exist; lint command is stale and type check has two known errors |
| Phaser world and tile map | Implemented | Layers, sprites, camera, resize, animation |
| Collision-aware local movement | Implemented | Arcade Physics across map collision layers |
| Multiplayer presence and movement | Implemented | Join snapshot, join/move/leave events, room relay |
| Smoothed/reconciled networking | Planned | Direct remote position application today |
| Server-validated movement | Planned | Client coordinates are trusted today |
| Room chat | Implemented | Live room broadcast and join/leave notices |
| Persistent chat/history | Planned | Browser-only chat state today |
| Live microphone/camera | Implemented | LiveKit capture, publish, subscribe, mute/camera controls |
| Screen sharing | Implemented | LiveKit screen publication + dedicated grid tile |
| Responsive meeting grid | Implemented | Participant/track event-driven tiles |
| Avatar-following local camera bubble | Implemented | Phaser coordinate/camera transform into DOM position |
| Proximity distance calculation | Implemented | Phaser dispatch every 100 ms |
| Proximity audio/video behavior | Partial | Consumers exist but are commented out; media is room-wide |
| Participant media-state list | Implemented | Combines LiveKit state with room coordination state |
| Owner handoff | Implemented | First socket owns; next connected socket inherits on exit |
| User team-role assignment | Implemented | Owner-validated Socket.IO mutation |
| Four role-labeled table zones | Implemented | Owner assignment, labels, matching-role welcome message |
| Durable room metadata | Modeled | Prisma `Room` exists; no current CRUD/join integration |
| Durable room membership/attendance | Modeled | Prisma `RoomMember` exists; live sockets do not use it |
| Capacity/privacy/password enforcement | Modeled | Fields exist; join flow accepts any non-empty room ID |
| Chat/screen-share/AFK room policy | Modeled | Fields exist; runtime does not enforce them |
| Single-container AWS deployment | Implemented | GitHub Actions, ECR, SSM, and EC2 Docker |
| Deployment health check/rollback | Planned | Workflow checks remote command status only |
| Horizontal application scaling | Planned | Live room state is single-process memory |
| Automated test suite | Planned | No unit/integration/end-to-end suite in the audited repository |

## Current engineering gaps

### Build health

The private checkout is live-deployed, but reproducibility checks are not green:

1. `npm run build` stops because Next.js detects both `src/middleware.ts` and `src/proxy.ts`.
2. `npm run lint` calls the removed `next lint` command.
3. standalone TypeScript checking finds an optional database URL passed where a string is required and a nullable OAuth password hash passed to bcrypt.
4. Next.js is configured to ignore TypeScript errors during its build, which should be removed after type cleanup.

### Room integrity and durability

- The room ID is currently the only live admission input.
- Socket identity begins from a client-supplied username rather than a server-bound authenticated session.
- Active players, owner, roles, and table assignments vanish on restart.
- Durable `Room` and `RoomMember` models are disconnected from Socket.IO.
- Room capacity/privacy/password and collaboration flags are not enforced.

### Security and abuse resistance

- The LiveKit token route needs authenticated membership checks.
- Real-time event payloads need systematic runtime validation and rate limiting.
- Movement needs bounds/speed validation if world rules become security-relevant.
- Signup avatar association needs stronger ownership verification.
- Reset email delivery and sensitive logging policy need production completion.

### Reliability and operations

- Deployment replaces one container and disconnects active rooms.
- No health endpoint, post-deploy smoke test, automatic rollback, or immutable image release is present.
- Local avatar files are not durable across image replacement.
- Automated tests and protocol/availability metrics are absent.

## Prioritized roadmap

### Phase 1 — Make the current system reproducible and safe

1. Consolidate route protection into `proxy.ts` and remove legacy middleware.
2. fix the two TypeScript errors, replace the lint command, and make build/lint/type checks required in CI.
3. authenticate LiveKit token requests and bind Socket.IO identity to the application session.
4. validate/rate-limit real-time events and stop logging sensitive token/reset material.
5. move avatar files to object storage.
6. add smoke coverage for auth, room join/move/leave, media-token authorization, and owner handoff.

### Phase 2 — Connect the durable room domain

1. add room create/read/update/join APIs around `Room` and `RoomMember`.
2. enforce creator/owner, privacy/password, capacity, chat, screen-share, and AFK policy.
3. reconcile durable identity with ephemeral socket presence.
4. define whether role/table state persists across sessions.
5. add attendance/history only with explicit privacy and retention rules.

### Phase 3 — Improve real-time quality

1. introduce movement sequence numbers, interpolation, validation, and measured update policy.
2. enable proximity media behind a tested feature flag with hysteresis and accessible controls.
3. unify cross-transport connection/recovery status in the room UI.
4. add moderation, reconnect, and duplicate-session behavior.
5. instrument latency, churn, room size, media failures, and event throughput.

### Phase 4 — Scale and harden delivery

1. add a shared Socket.IO adapter and external live-room state.
2. run multiple health-checked application replicas.
3. introduce graceful connection drain and rolling/blue-green delivery.
4. use immutable image tags and automated rollback.
5. add backup/restore drills, load tests, SLOs, alerts, and capacity targets.

## Product direction enabled by the architecture

Once durable room admission and shared live state are connected, the existing separation supports:

- persistent team spaces with enforceable policy;
- spatial audio/video zones;
- larger rooms through interest-based state updates;
- attendance and room analytics with explicit consent/retention;
- multiple maps or asset packs without changing the media plane;
- independent scaling of web/API, coordination, database, and media systems.

These are directions, not claims about the current release.
