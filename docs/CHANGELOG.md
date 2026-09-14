# Engineering change log

This public change log summarizes architecture and engineering updates without publishing implementation code. It complements the current-state matrix, which describes the resulting system rather than the sequence of changes.

## 15 September 2026: realtime security, correctness, testing, and performance

Private implementation commits: `caf2088`, `4b0c32e`, and `872b6e7`.

### Build and type safety

- Removed the empty legacy `src/middleware.ts`; `src/proxy.ts` is now the single Next.js 16 route-protection entry point.
- removed `typescript.ignoreBuildErrors` from the Next.js configuration.
- corrected Prisma database-URL typing through Prisma's typed environment helper.
- narrowed nullable OAuth-account password hashes before bcrypt comparison.
- added explicit `typecheck`, `test`, focused socket-test, and socket-benchmark scripts.

Result: standalone type-check and the Next.js 16.2.3 production build pass.

### Socket.IO authentication and protocol hardening

- Added connection middleware that requires a valid application access-token cookie.
- stores the verified JWT identity on the socket and derives the room username from it.
- stores the joined room on the server-side socket; movement, chat, and role events no longer trust a client-supplied room ID.
- removed the client-supplied chat username as an authority source.
- validates room IDs through the same bounded rule used by media authorization.
- rejects non-finite movement coordinates.
- trims chat messages, rejects empty content, and enforces a 2,000-character server limit.
- restricts participant roles, table roles, and table IDs through explicit allowlists.
- retains server-side owner checks and prevents ordinary assignment of `Room Owner` to another participant.

### LiveKit token hardening

- Requires a valid application access-token cookie.
- rejects a requested username that differs from the JWT identity.
- validates the requested room ID.
- binds the signed LiveKit identity to the authenticated username.
- extracts authorization into a focused server module used by tests.

Persisted room membership is not yet enforced because live rooms remain independent of the Prisma `Room` and `RoomMember` path.

### Room lifecycle corrections

- Extracted the realtime room engine from `server.ts` so production, tests, and benchmarks use the same handler.
- unified explicit leave and disconnect cleanup.
- corrected player removal, room-isolated notifications, empty-room deletion, and owner handoff.
- ensures the next owner receives the `Room Owner` role.
- replaced the redundant greeting and `existing-players` events with one `room-state-update` source.
- fixed an initialization race by installing the active room-state listener before joining.

### Movement and rendering performance

- Added a shared movement-policy module.
- limits moving clients to 20 snapshots per second instead of display-rate network emission.
- sends a final `turn` snapshot immediately when movement stops.
- keeps local rendering display-frame-rate driven.
- interpolates remote avatars with a bounded frame-rate-independent factor.
- snaps corrections larger than 200 world units.
- removed the unused 100 ms avatar-distance calculation and event dispatch.

Measured in the repeated 15-client local synthetic scenario, movement events and fan-out fell 66.7% and median average server CPU fell 45.5%, while p95 sender-to-recipient latency remained approximately 1.2 ms with zero delivery defects. See [Verification and benchmarks](VERIFICATION_AND_BENCHMARKS.md).

### Chat and participant interface

- Caps in-browser chat history at 300 messages.
- uses one animation-frame-scheduled instant scroll after message insertion.
- debounces participant-list reconstruction during LiveKit event bursts.
- indexes room players by username once per participant rebuild.
- replaces two-second participant-count polling with LiveKit connect/disconnect events.
- removes room IDs from role-change payloads because the server owns joined-room state.

### Server overhead and measurement

- Tries WebSocket first while retaining Socket.IO polling fallback and alternate-transport compatibility.
- removes duplicate initialization and greeting traffic.
- makes detailed realtime counters optional so production does not increment benchmark-only metrics.
- uses the Socket.IO adapter's room size for fan-out accounting when metrics are enabled.
- preserves benchmark sequence/timestamp metadata only in measurement workloads.

### Automated verification

- Added three movement-policy unit tests.
- added three LiveKit authorization tests.
- added four Socket.IO integration tests for unauthorized connections, room-isolated movement/chat, explicit leave, disconnect cleanup, and owner handoff.
- added an authenticated TypeScript load harness with separate server/generator telemetry, predeclared thresholds, delivery-integrity tracking, raw artifacts, capacity steps, repeated scenarios, and a ten-minute soak.

Result: all 10 focused tests pass. All optimized official benchmark runs pass their declared validity and workload thresholds.

### Dependency and repository cleanup

Repository-wide import checks confirmed seven unused direct packages before removal: Axios, Mongoose, body-parser, form-data, glob, js-yaml, and Motion. Framer Motion remains the animation dependency. The cleanup removed 33 package-lock entries.

The stale compiled `dist/server.js` artifact was removed and `dist` is ignored. Docker continues to compile the custom server during image creation. An unused background image remains as source artwork but is not downloaded during normal application rendering.

### Compatibility notes

- Realtime connections now require the application's HTTP-only access cookie.
- legacy clients relying on the greeting or `existing-players` events must consume `room-state-update`.
- chat and role clients no longer send authoritative username or room ID fields.
- active room state remains process-local and is cleared during container replacement.
- a push to private `main` triggers the deployment workflow, but a trigger alone is not proof that production deployment completed.

### Deferred work

- Broad API-route and browser end-to-end coverage.
- WebRTC/media-quality and browser-rendering benchmarks.
- EC2 or staging load tests and production-capacity claims.
- Direct ESLint migration for the stale `next lint` script.
- Redis/shared Socket.IO state and multiple application replicas.
- Database-backed room admission and room-policy enforcement.
- Proximity-based media behavior.
- Deployment health checks and automatic rollback.
