# Architecture decisions and tradeoffs

This record explains why the present Social Square architecture looks the way it does. It describes current choices, their benefits, and the pressure that would justify changing them.

## Socket.IO instead of raw WebSockets

**Decision:** Use Socket.IO rooms and event-based messages for multiplayer coordination.

**Why:** Built-in connection lifecycle, reconnection behavior, room addressing, and a clear named-event API reduced the amount of transport machinery required for the MVP.

**Cost:** Protocol overhead and a Socket.IO-specific client/server contract. Scaling to multiple Node replicas also requires a shared adapter.

**Revisit when:** Event volume or latency measurement shows transport overhead is material, or the system needs a protocol shared with non-Socket.IO clients.

## LiveKit SFU instead of peer-to-peer mesh

**Decision:** Route audio/video/screen media through LiveKit's Selective Forwarding Unit.

**Why:** In a browser mesh, each participant sends media separately to every peer, so upload cost grows with room size. An SFU gives each participant a much more stable connection shape and provides track subscription, screen share, adaptive streaming, dynacast, and reconnection primitives.

**Cost:** An external real-time dependency, token-signing integration, and hosted-service cost/operations.

**Revisit when:** Compliance, media-region control, or economics require a self-hosted LiveKit deployment—not to return to a large-room mesh.

## Phaser for the world, React for product UI

**Decision:** Keep game-loop rendering and physics in Phaser while panels/forms/navigation stay in React.

**Why:** Phaser is suited to sprites, tile maps, collisions, camera transforms, and keyboard updates. React is suited to authentication, forms, panels, track grids, and application navigation.

**Cost:** An integration boundary between declarative React state and an imperative scene. Browser events and refs must coordinate the two lifecycles.

**Revisit when:** Shared state grows enough to warrant a typed event bus or a dedicated room-session controller outside both rendering systems.

## Custom Node server and hybrid monolith

**Decision:** Serve Next.js, route handlers, Socket.IO, and LiveKit token issuance from one Node listener/container.

**Why:** Same-origin cookies and sockets, a single deployment artifact, and fast iteration are valuable at MVP scale. Long-lived WebSockets need a server process that can own them.

**Cost:** The application cannot use every purely serverless Next.js hosting pattern. WebSocket memory couples sessions to one process, and a container restart affects pages and room coordination together.

**Revisit when:** Independent scaling/release requirements become more expensive than monolith simplicity. The likely first extraction is real-time coordination, not ordinary page rendering.

## Mixed durable and ephemeral state

**Decision:** Persist identity/session/profile data in PostgreSQL while retaining live rooms in process memory.

**Why:** Durable identity requires transactional storage. Active movement and presence are short-lived and faster to iterate on without a persistence layer. Empty rooms can be reclaimed immediately.

**Cost:** Deployments erase rooms, horizontal replicas cannot share state, and modeled room policy is not enforced by the live path.

**Revisit when:** Persistent rooms, moderation, attendance, resume-after-failure, or more than one application instance become product requirements.

## Movement authority

**Decision:** The browser simulates local movement/collision; the server stores and relays reported coordinates.

**Why:** Immediate local feedback and a small implementation surface suit a collaborative workspace MVP where competitive cheating is not the primary threat.

**Cost:** A modified client can report impossible coordinates or speeds. Remote motion can jump because there is no reconciliation/interpolation protocol.

**Revisit when:** World rules, access zones, competitive mechanics, moderation, or unreliable-network quality require server validation/simulation.

## Separate media and coordination planes

**Decision:** Socket.IO state and LiveKit media share a logical room identifier but remain independent sessions.

**Why:** Small state events and high-bandwidth media have different transport, scaling, and recovery needs. The application can evolve multiplayer state without proxying media.

**Cost:** Identity, admission, disconnect, and recovery must stay consistent across two systems. A user can experience partial connection: game without media or media without complete room state.

**Revisit when:** Not the separation itself, but the session coordinator. A unified room-session state machine can make partial failures clearer without combining transports.

## Room owner as first connected socket

**Decision:** The first participant in a new live room becomes owner; ownership passes to another connected participant on departure.

**Why:** No extra setup is required and every room has someone capable of assigning coordination roles/tables.

**Cost:** Ownership is tied to transport order, not durable identity or creator policy. Reconnect order can change authority.

**Revisit when:** Durable room creation is wired into the live path. Ownership should then derive from stored room policy and authenticated user identity.

## JWT cookies with database-backed refresh allowlist

**Decision:** Use a short-lived access JWT plus a longer refresh JWT stored in PostgreSQL, both delivered through HTTP-only cookies.

**Why:** Access checks stay inexpensive while refresh sessions remain revocable. HTTP-only cookies reduce direct exposure to browser JavaScript.

**Cost:** Refresh records need cleanup/rotation policy, cookies require careful CSRF/origin policy, and the client still needs a complete refresh/retry experience.

**Revisit when:** Session-device management, refresh rotation, or an external identity provider becomes necessary.

## Local avatar files for the MVP

**Decision:** Validate uploads and write them into the container's public directory.

**Why:** Minimal infrastructure and immediate static serving.

**Cost:** Replacement images are ephemeral, multiple replicas would disagree, and the app container owns user content it cannot durably preserve.

**Revisit when:** Before relying on uploads in production. The next implementation should use object storage with immutable keys and a CDN.

## Mutable image tag and replacement deploy

**Decision:** Publish `latest` and replace the single running container through AWS Systems Manager.

**Why:** Straightforward, auditable deployment without exposing SSH as the CI transport.

**Cost:** Brief downtime, connection loss, weak rollback identity, and no automatic health-based rollback.

**Revisit when:** Reliability expectations require immutable releases, health probes, and rolling/blue-green deployment.
