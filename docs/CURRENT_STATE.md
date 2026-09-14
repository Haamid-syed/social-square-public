# Current state and roadmap

This capability matrix was verified against private `main` commit `872b6e7` on **15 September 2026**. It separates working behavior from partial, modeled, and planned work.

## Status definitions

| Status | Meaning |
|---|---|
| **Implemented** | Connected to the current user/runtime path |
| **Partial** | Meaningful implementation exists, but a material piece is missing |
| **Modeled** | Schema or UI concept exists, but is not connected to live behavior |
| **Planned** | Architectural direction without a complete current implementation |

## Capability matrix

| Capability | Status | Current evidence or boundary |
|---|---|---|
| Email/password signup and login | Implemented | Validation, hashing, account creation/lookup, cookies |
| Google and GitHub OAuth | Implemented | Authorization-code start/callback routes with state validation |
| Access and refresh sessions | Implemented | 15-minute access, 7-day database-backed refresh, logout revocation |
| Automatic access-token refresh UX | Partial | Refresh endpoint exists; full browser retry loop does not |
| Password reset | Partial | Secure token/reset path exists; transactional email does not |
| Student/remote-worker onboarding | Implemented | Profile branch and explicit completion flag |
| Profile/settings/password change | Implemented | Authenticated handlers and UI |
| Avatar uploads | Partial | Type/size validation and association work; storage is container-local and signup ownership path needs hardening |
| Protected web routes | Implemented | Single Next.js proxy, Edge JWT verification, onboarding guard |
| Socket.IO authentication | Implemented | Access-cookie middleware rejects invalid sessions and stores verified identity |
| LiveKit token authentication | Implemented | Session required; identity mismatch and invalid room ID rejected |
| Durable room authorization | Modeled | `Room`/`RoomMember` exist but do not control live admission |
| Production build | Implemented | Next.js 16.2.3 build passes; 27 routes/pages generated or registered |
| Standalone type-check | Implemented | `npm run typecheck` passes without build-error suppression |
| Lint command | Partial | Script still calls removed `next lint` and needs direct ESLint migration |
| Focused automated tests | Implemented | 10/10 movement, media-auth, and Socket.IO tests pass |
| Phaser world and tile map | Implemented | Layers, sprites, camera, resize, animation |
| Collision-aware local movement | Implemented | Arcade Physics across configured map layers |
| Multiplayer presence and movement | Implemented | Authenticated join, state snapshot, movement, leave, room relay |
| Movement throttling | Implemented | At most 20 snapshots/second plus immediate final stop |
| Remote movement interpolation | Implemented | Frame-rate-independent interpolation and large-gap snap |
| Server movement validation | Partial | Finite numbers and membership checked; speed/map/collision legality are not |
| Ordinary movement sequencing | Planned | Benchmark envelope supports measurement only; browser payloads do not add sequence metadata |
| Room chat | Implemented | Authenticated room binding, verified sender, trimming, 2,000-character server limit |
| Browser chat memory control | Implemented | History capped at 300 messages; animation-frame scroll |
| Persistent chat/history | Planned | Browser-only current history; no replay for new participants |
| Live microphone/camera | Implemented | LiveKit capture, publish, subscribe, and controls |
| Screen sharing | Implemented | LiveKit screen publication and dedicated grid tile |
| Responsive meeting grid | Implemented | Participant/track event-driven tiles |
| Avatar-following local camera bubble | Implemented | Phaser coordinate/camera transform into DOM position |
| Proximity audio/video | Planned | Distance calculation and event dispatch removed; dormant handlers remain disabled |
| Participant media-state list | Implemented | LiveKit and room state combined; rebuilds debounced and indexed |
| Participant count | Implemented | LiveKit connect/disconnect events instead of polling |
| Owner handoff | Implemented | Shared leave/disconnect cleanup; remaining participant inherits ownership |
| User and table roles | Implemented | Owner checks plus explicit role/table allowlists |
| Durable room metadata | Modeled | Prisma `Room` exists; no current CRUD/join integration |
| Durable room membership/attendance | Modeled | Prisma `RoomMember` exists; live sockets do not use it |
| Capacity/privacy/password enforcement | Modeled | Fields exist; live join does not enforce them |
| Chat/screen-share/AFK room policy | Modeled | Fields exist; runtime does not enforce them |
| Reproducible Socket.IO benchmark | Implemented | Authenticated harness, predeclared thresholds, 18 raw artifacts, integrity/resource telemetry |
| Single-container AWS deployment | Implemented | GitHub Actions, ECR, SSM, and EC2 Docker |
| Deployment health check/rollback | Planned | Workflow checks remote command status only |
| Horizontal application scaling | Planned | Live room state remains single-process memory |
| Broad API/browser/media test coverage | Planned | Current suite is intentionally focused |

## Delivered engineering changes

### Build and repository health

- Removed the empty legacy middleware, leaving `src/proxy.ts` as the single Next.js 16 route boundary.
- removed TypeScript build-error suppression.
- corrected Prisma environment typing and nullable OAuth-password handling.
- added explicit type-check, test, Socket.IO test, and benchmark scripts.
- removed seven confirmed-unused direct dependencies and 33 transitive lockfile entries.
- removed a stale compiled server artifact and ignored future `dist` output.

### Realtime security and correctness

- Bound Socket.IO and LiveKit identity to a verified application access cookie.
- stopped trusting client-supplied usernames and room IDs for chat, movement, and role operations.
- added room-ID, coordinate, chat, table, and role validation.
- unified explicit leave and disconnect cleanup.
- corrected room isolation and owner handoff.
- replaced redundant greeting/initial-player events with one `room-state-update` source.
- fixed the initial room-state listener race.

### Performance and interface behavior

- Reduced movement network frequency from display-rate emission to 20 Hz snapshots.
- added an immediate final stop snapshot and remote interpolation.
- removed unused avatar-distance calculations.
- capped browser chat history at 300 messages and replaced queued smooth scrolls with one scheduled instant scroll.
- debounced participant rebuilds and indexed username lookup.
- replaced two-second participant-count polling with LiveKit events.
- made benchmark-only realtime metrics optional in normal production handling.

### Evidence

- Added three movement unit tests, three LiveKit authorization tests, and four Socket.IO integration tests.
- added a reproducible authenticated load harness with separate server/generator telemetry.
- recorded baseline, capacity, optimized, ten-minute soak, and harness-validation artifacts against predeclared thresholds.

See [Verification and benchmarks](VERIFICATION_AND_BENCHMARKS.md) for exact results.

## Remaining engineering gaps

### Room integrity and durability

- Any authenticated user can enter or request media for a syntactically valid room ID.
- Active players, owner, roles, table assignments, and chat vanish on process restart.
- Durable `Room` and `RoomMember` models remain disconnected from Socket.IO and LiveKit admission.
- Capacity, privacy/password, and collaboration policy fields are not enforced.

### Security and abuse resistance

- Connection/event rate limiting is not yet documented as implemented.
- Movement accepts finite coordinates but does not enforce speed or map boundaries.
- Signup avatar association still needs stronger ownership verification.
- Password-reset email delivery and sensitive logging policy need production completion.

### Reliability and operations

- Deployment replaces one container and disconnects active rooms.
- No application health endpoint, post-deploy smoke test, automatic rollback, or immutable image release is present.
- Local avatar files are not durable across image replacement.
- The lint script remains stale.
- Browser end-to-end, broad API, media-quality, staging load, and deployment tests are not present.

## Prioritized roadmap

### Phase 1: complete the current production baseline

1. Replace `next lint` with direct ESLint invocation and enforce build, type-check, lint, and tests in CI.
2. connect Socket.IO and LiveKit admission to durable room membership and policy.
3. add rate limiting and origin policy for realtime and token routes.
4. move avatar files to object storage and tighten signup upload ownership.
5. add HTTP, database, WebSocket, and media-token deployment smoke checks.

### Phase 2: connect the durable room domain

1. Add room create/read/update/join APIs around `Room` and `RoomMember`.
2. enforce creator/owner, privacy/password, capacity, chat, screen-share, and AFK policy.
3. reconcile durable identity with ephemeral socket presence.
4. define whether roles and table assignments persist across sessions.
5. add attendance/history only with explicit privacy and retention rules.

### Phase 3: extend realtime quality

1. Add ordinary movement sequence numbers and stronger reconnect/reconciliation semantics if measurements justify them.
2. enforce movement speed and map bounds where position becomes security-relevant.
3. implement proximity media behind a tested feature flag with hysteresis and accessible controls.
4. unify Socket.IO and LiveKit connection/recovery state in the room UI.
5. add browser, WebRTC, adverse-network, and staging benchmarks.

### Phase 4: scale and harden delivery

1. Add a shared Socket.IO adapter and external live-room state.
2. run multiple health-checked application replicas.
3. introduce graceful connection drain and rolling or blue/green delivery.
4. use immutable image tags and automated rollback.
5. add backup/restore drills, load tests, service objectives, alerts, and capacity targets.

## Product direction

Once durable room admission and shared live state are connected, the architecture can support persistent team spaces, enforced room policy, spatial media zones, interest-based updates, consent-based attendance analytics, multiple maps, and independent scaling of application, coordination, data, and media services.

These are roadmap directions, not claims about the current release.
