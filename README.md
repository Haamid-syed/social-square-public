<h1 align="center">Social Square</h1>

<p align="center">
  A real-time, game-like virtual workspace where people move through a shared 2D room,<br />
  meet over live audio/video, share screens, chat, and organize around team tables.
</p>

<p align="center">
  <a href="https://socialsquare.tech"><strong>Live application</strong></a>
  ·
  <a href="https://www.youtube.com/watch?v=hMgD_D21UHY"><strong>Video demo</strong></a>
  ·
  <a href="docs/README.md"><strong>Engineering blueprint</strong></a>
  ·
  <a href="https://github.com/Haamid-syed/Social-Square"><strong>Private implementation</strong></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Next.js-16-black?logo=next.js" alt="Next.js 16" />
  <img src="https://img.shields.io/badge/React-19-61DAFB?logo=react" alt="React 19" />
  <img src="https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript" alt="TypeScript 5" />
  <img src="https://img.shields.io/badge/Phaser-3.90-8A2BE2" alt="Phaser 3.90" />
  <img src="https://img.shields.io/badge/LiveKit-WebRTC-00B4D8" alt="LiveKit WebRTC" />
  <img src="https://img.shields.io/badge/Socket.IO-4-010101?logo=socket.io" alt="Socket.IO 4" />
  <img src="https://img.shields.io/badge/PostgreSQL-Prisma-336791?logo=postgresql" alt="PostgreSQL and Prisma" />
  <img src="https://img.shields.io/badge/AWS-EC2%20%2B%20ECR-FF9900?logo=amazonaws" alt="AWS EC2 and ECR" />
</p>

> [!IMPORTANT]
> This is the public, documentation-only repository for Social Square. It explains the product and its engineering in depth, but intentionally contains no application source code, credentials, private infrastructure identifiers, or deployable build artifacts. The implementation remains private and is not offered under an open-source license.

## Current status

**Social Square is a working, deployed MVP at [socialsquare.tech](https://socialsquare.tech).** The production application is containerized on AWS, persists account and profile data in managed PostgreSQL, and uses LiveKit Cloud for real-time media. This blueprint was verified against the private implementation on **14 September 2026**.

| Area | Current state |
|---|---|
| Identity | Email/password, Google OAuth, GitHub OAuth, access/refresh cookies |
| User experience | Onboarding, editable profiles, avatar uploads, account settings |
| Virtual rooms | Phaser tile map, synchronized avatars, collision-aware movement |
| Collaboration | Live audio/video, meeting grid, screen share, room chat |
| Coordination | Participant list, room-owner handoff, user roles, table-role labels |
| Durable data | Users, profiles, refresh tokens, reset tokens, and room-related schema |
| Ephemeral data | Active players, ownership, roles, table assignments, and chat |
| Delivery | GitHub Actions to AWS ECR to AWS Systems Manager to Docker on EC2 |

The implementation already calculates avatar-to-avatar distance, but proximity-based media attenuation and visibility are not enabled in the current release. Audio and video are room-wide today. See [Current state and roadmap](docs/CURRENT_STATE.md) for an exact implemented/partial/planned breakdown.

## What the product does

Social Square replaces the flat meeting grid with a navigable shared place. An authenticated user joins a room as an animated avatar, moves through a collision-aware pixel workspace, sees other participants move in real time, and opens media, chat, and coordination tools without leaving the world.

The primary user flow is:

1. Sign up with email/password or continue with Google/GitHub.
2. Complete a student or remote-worker onboarding profile.
3. Enter a room identifier from the join screen.
4. Load the Phaser world, Socket.IO session, and LiveKit session together.
5. Move with WASD/arrow keys and collaborate through audio, video, screen share, chat, and the participant panel.
6. Use team roles and four assignable table zones to organize the room.
7. Leave explicitly or disconnect; the server removes the player and hands ownership to another participant when required.

## System at a glance

```mermaid
flowchart LR
    User["Browser user"]

    subgraph Browser["Browser runtime"]
        UI["Next.js + React UI"]
        World["Phaser world"]
        Realtime["Socket.IO client"]
        Media["LiveKit client"]
        UI <--> World
        UI <--> Realtime
        UI <--> Media
    end

    subgraph Host["AWS EC2 · Docker container"]
        HTTP["Express + Next.js\nHTTP and route handlers"]
        Socket["Socket.IO server\nactive room state"]
        Token["LiveKit token endpoint"]
    end

    DB["Managed PostgreSQL\nusers, profiles, tokens"]
    SFU["LiveKit Cloud\nWebRTC SFU"]
    CI["GitHub Actions"]
    Registry["AWS ECR"]

    User --> Browser
    UI <--> HTTP
    Realtime <--> Socket
    Media <--> SFU
    Token --> SFU
    HTTP <--> DB
    CI --> Registry --> Host
```

One custom Node process owns the application HTTP listener. It prepares Next.js, mounts Socket.IO on the same server, exposes the LiveKit token route, and delegates remaining traffic to Next.js. Media packets do not travel through the application server; LiveKit handles that separate data plane.

## Core capabilities

### Spatial multiplayer

- 2D Tiled workspace rendered with Phaser 3 and Arcade Physics.
- WASD/arrow-key movement, directional animation, collisions, camera follow, pixel rendering, responsive resize, and adaptive zoom.
- Socket.IO synchronization for joining, existing players, movement, disconnection, ownership, roles, and table assignments.
- Four table zones whose labels can be assigned by the room owner; matching participants receive an in-world welcome state.

### Live collaboration

- LiveKit audio/video rooms with local publishing, automatic remote subscription, adaptive streaming, dynacast, and reconnect handling.
- Microphone, camera, and screen-share controls with a responsive meeting grid.
- A floating local-camera bubble that tracks the local avatar while the meeting panel is closed.
- Room chat, join/leave notices, participant media states, and owner-controlled coordination roles.

### Accounts and profiles

- Password authentication with validation and bcrypt hashing.
- Google and GitHub OAuth authorization-code flows with state validation.
- Short-lived access tokens and database-backed refresh tokens in HTTP-only cookies.
- Student/remote-worker onboarding, profile editing, avatar upload, password change, and password-reset token flow.
- Protected application routes and an onboarding guard.

## Engineering blueprint

| Document | What it answers |
|---|---|
| [Architecture](docs/ARCHITECTURE.md) | What runs where, how the major subsystems connect, and where state lives |
| [Component map](docs/COMPONENT_MAP.md) | Which important private-repository files own each responsibility and how they depend on one another |
| [Real-time protocol](docs/REALTIME_PROTOCOL.md) | Socket.IO contracts, room lifecycle, ownership, movement, chat, roles, and LiveKit separation |
| [Data, auth, and API](docs/DATA_AUTH_API.md) | Persistent models, token lifecycle, route protection, uploads, and the HTTP surface |
| [Deployment and operations](docs/DEPLOYMENT_OPERATIONS.md) | Container topology, CI/CD path, configuration boundaries, failure modes, and operational gaps |
| [Design decisions](docs/DESIGN_DECISIONS.md) | Why the project uses Socket.IO, LiveKit, Phaser, a custom server, and mixed durable/ephemeral state |
| [Current state and roadmap](docs/CURRENT_STATE.md) | What is implemented, partial, modeled only, or still planned |
| [Full architecture map](docs/FULL_ARCHITECTURE_MAP.md) | Compact end-to-end reference for the entire system |

## Technology stack

| Layer | Technologies | Responsibility |
|---|---|---|
| Web application | Next.js 16 App Router, React 19, TypeScript | Pages, route handlers, layouts, and integrated room UI |
| World simulation | Phaser 3, Tiled assets, Arcade Physics | Rendering, camera, animation, movement, collisions, spatial zones |
| Real-time state | Socket.IO client/server | Presence, movement, chat, ownership, roles, table assignments |
| Media | LiveKit client/server SDK, WebRTC SFU | Audio, video, screen sharing, subscriptions, reconnects |
| Client state | Zustand | Auth/user state shared across React surfaces |
| Design system | Tailwind CSS, Radix UI, shadcn-style components, Framer Motion | Responsive, accessible interaction layer |
| Server | Node.js 20, Express 5, Next.js route handlers | Shared listener, APIs, sockets, media-token issuance |
| Data | PostgreSQL, Prisma 6 | Accounts, profiles, durable tokens, room schema |
| Security | Zod, bcryptjs, JWT via jsonwebtoken/jose | Validation, hashing, Node/Edge token verification |
| Infrastructure | Docker, GitHub Actions, AWS ECR/EC2/SSM, LiveKit Cloud | Build, registry, deployment, hosting, managed media |

## Engineering scope and authorship

Social Square was designed and implemented end-to-end by [Haamid Syed](https://github.com/Haamid-syed). The work spans product UI, multiplayer state synchronization, Phaser world behavior, Socket.IO room lifecycle, LiveKit media integration, authentication and profiles, PostgreSQL modeling, and container delivery on AWS.

Key engineering challenges included:

- reconciling browser, Phaser, Socket.IO, and LiveKit lifecycles on one room screen;
- cleaning up correctly through refreshes, explicit exits, network drops, and owner departure;
- keeping media on a scalable SFU path while spatial state uses small real-time events;
- separating durable account data from high-churn live-room data;
- deploying WebSocket and WebRTC-aware behavior behind HTTPS; and
- preserving a path from the current single-process MVP to shared state and multiple replicas.

## Boundaries and ownership

This repository is intended for portfolio review, engineering discussion, and architecture evaluation. It is not a runnable substitute for the private repository. No source license is granted by publication of these documents; see [LICENSE](LICENSE).

Read-only access to the private implementation can be considered for interview or formal evaluation purposes. Please contact the repository owner through GitHub.
