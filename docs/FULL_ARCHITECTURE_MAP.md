# Social Square full architecture map

This document provides a compact end-to-end reference. The focused documents provide additional rationale and implementation status.

## Complete system map

```mermaid
flowchart TB
    subgraph Browser["Browser runtime"]
        Public["Landing + FAQ"]
        AuthPages["Login · signup · OAuth · reset"]
        Account["Onboarding · profile · settings"]
        Join["Join-room UI"]
        Store["Zustand auth/user store"]

        subgraph LiveRoom["/room/[roomId]"]
            Orchestrator["Room page orchestrator"]
            Canvas["GameCanvas lifecycle bridge"]
            Scene["Phaser main scene"]
            SocketClient["Socket.IO client"]
            LKClient["LiveKit client"]
            Toolbar["Bottom toolbar"]
            Chat["Room chat"]
            People["Participant + role panel"]
            Grid["Meeting / screen-share grid"]
            Bubble["Avatar-following local video"]

            Orchestrator --> Canvas --> Scene
            Canvas --> SocketClient
            Canvas --> LKClient
            Orchestrator --> Toolbar
            Orchestrator --> Chat
            Orchestrator --> People
            Orchestrator --> Grid
            Orchestrator --> Bubble
        end

        Public --> AuthPages --> Account --> Join --> LiveRoom
        Store <--> AuthPages
        Store <--> Account
        Store <--> LiveRoom
    end

    subgraph AppContainer["Docker application container"]
        Entry["server.ts · custom Node entry point"]
        Express["Express + HTTP server"]
        Next["Next.js pages and route handlers"]
        SocketServer["Authenticated Socket.IO\nsocketServer.ts"]
        MediaToken["Authenticated LiveKit grant\nlivekitAuth.ts"]
        Rooms["In-memory rooms"]

        Entry --> Express
        Express --> Next
        Express --> SocketServer
        Express --> MediaToken
        SocketServer <--> Rooms
    end

    subgraph Data["Managed data services"]
        Prisma["Prisma client"]
        Postgres["PostgreSQL"]
        Users["Users + profiles"]
        Tokens["Refresh/reset tokens"]
        RoomModels["Room/member schema\nnot wired to live join"]
        Prisma --> Postgres
        Postgres --> Users
        Postgres --> Tokens
        Postgres --> RoomModels
    end

    subgraph MediaService["LiveKit Cloud"]
        SFU["WebRTC SFU"]
        Tracks["Mic · camera · screen tracks"]
        SFU <--> Tracks
    end

    subgraph Delivery["Delivery path"]
        Git["Private source · main"]
        Actions["GitHub Actions"]
        ECR["AWS ECR image"]
        SSM["AWS Systems Manager"]
        EC2["AWS EC2 Docker host"]
        Git --> Actions --> ECR
        Actions --> SSM --> EC2
        ECR --> EC2
    end

    subgraph Evidence["Verification path"]
        Tests["10 focused tests"]
        Harness["Authenticated load harness"]
        Results["Thresholds + 18 artifacts"]
        Tests --> SocketServer
        Tests --> MediaToken
        Harness --> SocketServer
        Harness --> Results
    end

    Browser <-->|HTTPS| Next
    SocketClient <-->|WebSocket / fallback| SocketServer
    LKClient <-->|WebRTC| SFU
    LKClient -->|HTTPS token request| MediaToken
    Next <--> Prisma
    EC2 --> AppContainer
```

## Runtime paths

### Authentication path

```text
Auth page
-> Next.js auth route
-> Zod validation
-> bcrypt / OAuth provider
-> Prisma + PostgreSQL
-> access and refresh HTTP-only cookies
-> route proxy + AuthProvider
-> onboarding guard
```

### Multiplayer path

```text
Room page
-> Socket.IO connect with access cookie
-> join-room
-> server validates ID and creates/loads in-memory room
-> room-state-update initial snapshot
-> Phaser remote sprites
-> 20 Hz movement snapshots relayed to joined-room peers
-> render-rate remote interpolation
-> leave/disconnect cleanup + owner handoff
```

### Media path

```text
Room page
-> authenticated application media-token endpoint
-> session, identity, and room validation
-> signed identity-bound LiveKit room grant
-> LiveKit SFU connection
-> local mic/camera publication
-> remote subscriptions
-> meeting grid / floating local camera
-> optional screen-share publication
```

### Role and table path

```text
Owner participant panel
-> assign-user-role or assign-table-role
-> server verifies current socket is room owner
-> in-memory room state mutation
-> room-state-update broadcast
-> participant panel refresh
-> browser CustomEvent
-> Phaser table label and zone behavior
```

### Deployment path

```text
Push to private main
-> GitHub Actions buildx
-> Linux AMD64 image in ECR
-> Systems Manager remote command
-> EC2 pulls image
-> old container replaced
-> custom Node server starts on port 3003
```

### Verification path

```text
Production realtime and media-authorization modules
-> movement and authorization unit tests
-> authenticated Socket.IO integration tests
-> isolated child-process load harness
-> predeclared validity and workload thresholds
-> baseline, optimized, capacity, soak, and validation artifacts
-> public scope-qualified result report
```

## Important private files

| Area | File | Responsibility |
|---|---|---|
| Process | `server.ts` | Next.js preparation, Express listener, server-module registration, LiveKit token signing |
| Realtime server | `src/server/socketServer.ts` | Socket authentication, validation, room registry, events, cleanup, optional metrics |
| Media authorization | `src/server/livekitAuth.ts` | Session, identity, and room validation before token signing |
| Room composition | `src/app/room/[roomId]/page.tsx` | Integrates game, sockets, media, panels, controls, and video bubble |
| Game lifecycle | `src/lib/game/gameCanvas.tsx` | Creates/cleans Phaser, Socket.IO, LiveKit, and media elements |
| Phaser setup | `src/lib/game/gameInit.ts` | Game configuration and teardown |
| World logic | `src/lib/game/gameScene.ts` | Map, physics, 20 Hz snapshots, remote interpolation, tables |
| Movement policy | `src/lib/game/movementSync.ts` | Shared send-rate, final-stop, and interpolation policy |
| Socket adapter | `src/lib/sockets/socketInit.ts` | Authenticated same-origin connection, join, initial state, scene mapping |
| Media connection | `src/lib/video/videoInit.ts` | Token fetch, LiveKit connect/retry, local publish, remote subscription |
| Media DOM | `src/lib/video/handleVideo.ts` | Media attachment/detachment and cleanup |
| Meeting UI | `src/components/VideoGrid.tsx` | Participant, camera, mute, and screen-share tiles |
| Chat UI | `src/components/RoomChat.tsx` | Capped current-session messages, join/leave notices, scheduled scroll |
| Coordination UI | `src/components/ParticipantsList.tsx` | Presence/media status and owner role/table controls |
| Controls | `src/components/BottomToolbar.tsx` | Media, panels, screen share, and leave actions |
| Client identity | `src/app/state/atoms.tsx` | Zustand auth/user state |
| Auth helpers | `src/lib/auth.ts`, `src/lib/auth-edge.ts` | Password/JWT behavior in Node and Edge runtimes |
| Validation | `src/lib/validations.ts` | Zod boundary schemas |
| Persistence | `prisma/schema.prisma` | User, token, room, and membership models |
| Access gate | `src/proxy.ts` | Protected/public route redirects |
| Container | `Dockerfile` | Multi-stage production image |
| Delivery | `.github/workflows/deploy.yml` | ECR build/push and SSM-driven EC2 replacement |
| Tests | `tests/` | Movement, media authorization, and realtime integration coverage |
| Benchmarks | `benchmarks/` | Authenticated load harness, thresholds, telemetry, and raw artifacts |

## State map

| State | Authority | Lifetime | Replicated how? |
|---|---|---|---|
| User/profile | PostgreSQL | Durable | Queried through Prisma |
| Refresh/reset token | PostgreSQL + signed token | Until expiry/revocation | HTTP cookie + database record |
| Durable room definition | PostgreSQL schema | Durable | Not used by current room path |
| Active room membership | Socket.IO server memory | Process/room lifetime | `room-state-update` snapshots |
| Position/animation | Local client then server registry | Live connection | 20 Hz `player-move` snapshots plus render interpolation |
| Owner/user roles/tables | Socket.IO server memory | Process/room lifetime | Server-validated state broadcasts |
| Chat | Each browser, maximum 300 messages | Mounted page lifetime | Live broadcast only, no replay |
| Camera/mic/screen | LiveKit | Media session | SFU publications/subscriptions |
| UI panel/control state | React component state | Mounted room page | Local only |

## Current architecture and next stage

| Concern | Current architecture | Next stage |
|---|---|---|
| Rooms | Authenticated users join syntactically valid ephemeral IDs | Durable, membership- and policy-enforced rooms |
| Scaling | One application process | Shared adapter/state and multiple replicas |
| Movement | Client physics, 20 Hz snapshots, finite-coordinate checks, remote interpolation | Speed/map validation and sequence-aware recovery where required |
| Proximity | Distance emission removed; media remains room-wide | Feature-flagged attenuation/visibility with UX controls |
| Chat | Live, browser-only | Optional durable history with moderation/retention policy |
| Uploads | Container-local | Object storage/CDN |
| Delivery | Mutable tag, container replacement | Immutable releases, health checks, rollback, drain |
| Quality gates | Build/type-check and 10 focused tests pass; lint script stale | Required CI gates plus broad API/browser/media coverage |

## Detailed references

- [System architecture](ARCHITECTURE.md)
- [Component map](COMPONENT_MAP.md)
- [Real-time protocol](REALTIME_PROTOCOL.md)
- [Data, authentication, and API](DATA_AUTH_API.md)
- [Deployment and operations](DEPLOYMENT_OPERATIONS.md)
- [Architecture decisions](DESIGN_DECISIONS.md)
- [Verification and benchmarks](VERIFICATION_AND_BENCHMARKS.md)
- [Current state and roadmap](CURRENT_STATE.md)
