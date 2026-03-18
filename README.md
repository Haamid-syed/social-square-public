(Note : This repo only contains the outline and system architecture of the project. I'll update the repo with the latest updates in the project soon)

Social Square - Real-Time Multiplayer Social Platform (Gather-Town-like)
Social Square is a browser-based multiplayer social environment where users can move around shared 2D spaces, see other players in real time, and communicate via proximity-based voice chat. The project focuses on low-latency real-time systems, room lifecycle management, and scalable WebRTC communication, rather than visual polish alone.

1. Problem Statement
Most social platforms optimize for feeds and messages, not presence. Social Square explores how to create a feeling of shared space on the web, where:
* Multiple users coexist in the same environment
* Player movement and state remain consistent across clients
* Voice communication scales beyond a few peers
* Users can freely join, leave, and switch rooms without breaking state
The core challenge is synchronizing real-time interaction reliably over an unreliable network.

2. Why This Is Hard
Building real-time multiplayer systems on the web introduces several non-trivial problems:
* State synchronization: Keeping player positions and actions consistent across clients with varying latency.
* Connection churn: Users disconnect abruptly, refresh pages, or switch rooms.
* Scalable voice chat: Peer-to-peer WebRTC does not scale well beyond small groups.
* Room lifecycle management: Avoiding ghost users, memory leaks, and stale rooms.
* Deployment constraints: Secure HTTPS is mandatory for WebRTC media access.
These challenges go beyond typical CRUD-based web applications.

3. System Architecture
High-Level Overview
* Frontend
    * Next.js, Shadcn, Framer Motion for UI
    * Phaser.js for 2D rendering and game-loop logic
* Backend
    * A seperate custom Node.js + Express
    * Socket.IO for real-time bidirectional communication
* Voice Communication
    * WebRTC via LiveKit (SFU architecture)
* Infrastructure
    * Deployed on AWS EC2
    * Nginx as reverse proxy
    * HTTPS via Let’s Encrypt (Certbot)

Real-Time Flow (Simplified)
1. Client connects via Socket.IO
2. User joins a room
3. Server:
    * Registers player
    * Sends existing player state
    * Broadcasts join event
4. Client emits movement updates
5. Server relays updates to room participants
6. On disconnect:
    * Player state is cleaned up
    * Room is destroyed if empty
Voice traffic is handled separately via an SFU to avoid N×N peer connections.

4. My Contributions
This project was designed and implemented end-to-end by me, including:
* Real-time multiplayer room architecture
* Player state synchronization logic
* Room creation, cleanup, and lifecycle handling
* Socket.IO event design and payload structure
* WebRTC voice integration using an SFU model
* Deployment on EC2 with HTTPS configuration
* Debugging network-level issues (CORS, HTTPS, private vs public IPs)
The focus was on system correctness and robustness, not just UI behavior.

5. Technical Learnings & Tradeoffs
* Socket.IO vs raw WebSockets: Socket.IO’s reconnection and room abstractions simplified state recovery at the cost of some overhead.
* SFU over mesh WebRTC: Required more setup but enabled scalable voice communication.
* Authoritative server model: Reduced cheating and divergence at the expense of slightly higher server responsibility.
* Monolith deployment: Faster iteration early on, with clear paths toward future service separation.

Scaling & Future Work
Potential next steps include:
* Horizontal scaling with Redis-backed Socket.IO adapters
* Interest-based updates (only syncing nearby players)
* Persistent user profiles and world state
* Fault-tolerant room recovery
* Metrics and latency observability

Note : Currently, Only the core features (game ui, sockets and webRTC logic) along with some frontend is implemented.

Demo
* Live deployment: https://www.socialsquare.tech/

Source Code Access
The full implementation is maintained in a private repository. Read-only access can be provided for interview or evaluation purposes upon request.
