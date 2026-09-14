# Verification and benchmark results

Last reconciled with the private implementation: 15 September 2026.

This report summarizes Social Square's focused automated tests and Socket.IO benchmarks. It publishes the methodology, acceptance criteria, measured results, and limitations without publishing application or load-harness source code.

## Executive summary

- All 10 focused automated tests passed.
- The standalone TypeScript check and Next.js 16.2.3 production build passed.
- Changing avatar movement from 60 Hz emissions to 20 Hz snapshots reduced movement events and recipient fan-out by 66.7% in the repeated 15-client scenario.
- Median average server CPU in that scenario fell 45.5%, from 19.7% to 10.7%; median p95 sender-to-recipient latency remained effectively flat at 1.12 ms versus 1.2 ms.
- The optimized 240-client, 16-room run sustained 67,207.5 recipient deliveries per second at 1.1 ms p95 and passed every predeclared threshold.
- Every official recorded run had zero missing deliveries, duplicate deliveries, and unexpected disconnects.

These results describe synthetic authenticated Socket.IO clients on local loopback. They are not production, browser-rendering, database, or WebRTC capacity claims.

## Verification summary

| Command or check | Result |
|---|---|
| `npm run typecheck` | Pass |
| `npm test` | Pass: 10/10 |
| `npm run build` | Pass: Next.js 16.2.3 production build; 27 routes/pages generated or registered |
| Docker server TypeScript compilation | Pass; server and imported modules emitted to a temporary directory |
| Repository whitespace check | Pass |
| Removed-dependency check | Pass; confirmed removed direct packages are absent |

These results are taken from the private repository's recorded verification report dated 15 September 2026.

The current `npm run lint` script still calls the removed `next lint` command. It is not represented as passing and remains documented technical debt.

## Automated test results

| Test | Type | Result |
|---|---|---|
| Movement snapshots are limited to 20 Hz | Unit | Pass |
| A final snapshot is sent immediately when movement stops | Unit | Pass |
| Remote interpolation is frame-rate independent and bounded | Unit | Pass |
| LiveKit authorization rejects requests without an authenticated cookie | Authorization | Pass |
| LiveKit authorization rejects identity spoofing | Authorization | Pass |
| LiveKit authorization binds media identity to the authenticated user | Authorization | Pass |
| Socket.IO rejects clients without a valid access token | Integration | Pass |
| Movement and chat are isolated by the socket's joined room | Integration | Pass |
| Explicit leave removes the player and hands ownership to a remaining participant | Integration | Pass |
| Owner disconnect performs the same owner handoff | Integration | Pass |

The integration tests start temporary localhost Socket.IO servers and use synthetic JWT identities. They do not use real user accounts, PostgreSQL, LiveKit, or external services.

## Benchmark scope

The harness starts an isolated child-process server using the same extracted realtime handler as production. Synthetic `socket.io-client` processes authenticate with signed access-token cookies, join real application rooms, and emit the application `player-move` protocol.

| Property | Recorded environment |
|---|---|
| Host | Apple M1, 8 logical CPUs, 8 GiB RAM |
| Operating system | macOS, arm64 |
| Runtime | Node.js v22.23.1 |
| Network | `127.0.0.1` loopback |
| Transport | Socket.IO over WebSocket |
| Authentication | Synthetic JWT `accessToken` cookies |
| Server under test | Isolated production realtime handler |
| PostgreSQL | Excluded |
| Next.js request handling | Excluded |
| LiveKit/WebRTC | Excluded |
| Browser rendering | Excluded |

The private evidence set contains 18 JSON artifacts covering baseline, optimized, capacity, soak, and harness-validation runs. Each current-format artifact records repository state, command, environment, implementation hashes, workload, latency histogram, delivery-integrity counters, server telemetry, generator telemetry, thresholds, and interpretation.

## Acceptance and validity criteria

Thresholds were declared before the official runs.

| Category | Threshold |
|---|---:|
| Sender-to-recipient p95 latency | At most 50 ms |
| Sender-to-recipient p99 latency | At most 100 ms |
| Missing recipient deliveries | 0 |
| Duplicate recipient deliveries | 0 |
| Unexpected disconnects | 0 |
| Average server CPU | At most 85% |
| Peak sampled server CPU | At most 95% |
| Peak server RSS | At most 256 MiB |
| Load-generator emission attainment | At least 95% |
| Average load-generator CPU | At most 85% |
| Load-generator event-loop p99 | At most 25 ms |

A run is valid only if the generator meets its attainment, CPU, and event-loop limits. A valid workload passes only if every latency, delivery-integrity, disconnect, and server-resource threshold also passes.

## Preserved 60 Hz baseline

The baseline reproduced the earlier frame-rate-driven workload at 60 movement updates per second for every continuously moving client.

| Workload | Duration | Incoming events/s | Deliveries/s | p95 | p99 | Missing / duplicate / disconnects | Result |
|---|---:|---:|---:|---:|---:|---:|---|
| 1 room x 2 clients | 10 s | 120.0 | 120.0 | 2.01 ms | 3.45 ms | 0 / 0 / 0 | Pass |
| 1 room x 5 clients | 10 s | 300.1 | 1,200.4 | 1.29 ms | 2.43 ms | 0 / 0 / 0 | Pass |
| 1 room x 10 clients | 10 s | 600.0 | 5,400.0 | 1.17 ms | 2.06 ms | 0 / 0 / 0 | Pass |
| 1 room x 15 clients, run 1 | 15 s | 900.1 | 12,600.9 | 1.12 ms | 2.39 ms | 0 / 0 / 0 | Pass |
| 1 room x 15 clients, run 2 | 15 s | 900.1 | 12,600.9 | 1.12 ms | 2.26 ms | 0 / 0 / 0 | Pass |
| 1 room x 15 clients, run 3 | 15 s | 898.9 | 12,584.1 | 2.79 ms | 3.63 ms | 0 / 0 / 0 | Pass |
| 2 rooms x 15 clients | 15 s | 1,799.7 | 25,196.3 | 1.07 ms | 1.77 ms | 0 / 0 / 0 | Pass |
| 4 rooms x 15 clients | 15 s | 3,579.5 | 50,112.5 | 4.24 ms | 7.85 ms | 0 / 0 / 0 | Pass |
| 8 rooms x 15 clients | 15 s | 7,183.8 | 100,573.2 | 2.99 ms | 5.09 ms | 0 / 0 / 0 | Pass |
| 16 rooms x 15 clients | 10 s | 14,247.7 | 199,467.8 | 14.66 ms | 37.49 ms | 0 / 0 / 0 | Fail: peak server CPU |

The 240-client baseline had a valid generator and met latency, integrity, disconnect, average CPU, and RSS limits. It failed only because sampled peak server CPU reached 118.5%, above the declared 95% limit. The highest passing 60 Hz workload tested was 120 simulated clients across eight rooms; that observation does not establish an exact capacity boundary.

### Repeated baseline median

| Metric, 1 room x 15 clients | Median |
|---|---:|
| Incoming events/s | 900.1 |
| Recipient deliveries/s | 12,600.9 |
| p95 / p99 | 1.12 ms / 2.39 ms |
| Average server CPU | 19.7% |
| Peak server RSS | 97.2 MiB |
| Delivery defects / disconnects | 0 / 0 |

## Ten-minute baseline soak

| Metric | Result |
|---|---:|
| Clients / rooms | 15 / 1 |
| Duration | 600 seconds |
| Server-handled events | 539,700 |
| Expected and observed recipient deliveries | 7,555,800 |
| Recipient deliveries/s | 12,593.0 |
| Latency p50 / p95 / p99 / max | 1.6 / 3.2 / 4.0 / 55.44 ms |
| Missing / duplicate / unexpected disconnects | 0 / 0 / 0 |
| Server CPU average / peak | 19.3% / 54.3% |
| Peak server RSS | 115.7 MiB |
| Generator emission attainment | 99.94% |
| Outcome | Valid and passing |

Server RSS did not grow monotonically during the run. That observation is not sufficient to claim the absence of a memory leak.

## Optimized 20 Hz protocol

The current browser protocol sends at most 20 position snapshots per second while moving, emits a final stop snapshot immediately, and interpolates remote avatars during rendering. The benchmark imports the same movement-rate constant used by the client.

| Workload | Duration | Incoming events/s | Deliveries/s | p95 | p99 | Server CPU average / peak | Missing / duplicate / disconnects | Result |
|---|---:|---:|---:|---:|---:|---:|---:|---|
| 1 room x 15 clients, run 1 | 15 s | 300.1 | 4,200.9 | 1.2 ms | 2.2 ms | 10.7% / 44.2% | 0 / 0 / 0 | Pass |
| 1 room x 15 clients, run 2 | 15 s | 300.0 | 4,200.0 | 1.0 ms | 1.3 ms | 10.2% / 25.4% | 0 / 0 / 0 | Pass |
| 1 room x 15 clients, run 3 | 15 s | 300.1 | 4,200.9 | 1.2 ms | 1.7 ms | 12.8% / 40.2% | 0 / 0 / 0 | Pass |
| 8 rooms x 15 clients | 15 s | 2,400.1 | 33,601.9 | 0.9 ms | 1.2 ms | 26.0% / 45.6% | 0 / 0 / 0 | Pass |
| 16 rooms x 15 clients | 15 s | 4,800.5 | 67,207.5 | 1.1 ms | 1.4 ms | 31.3% / 60.7% | 0 / 0 / 0 | Pass |

Every optimized run was valid and passed every declared workload threshold.

## Baseline versus optimized

| Metric, 1 room x 15 clients | 60 Hz baseline median | 20 Hz optimized median | Change |
|---|---:|---:|---:|
| Incoming movement events/s | 900.1 | 300.1 | -66.7% |
| Recipient deliveries/s | 12,600.9 | 4,200.9 | -66.7% |
| Sender-to-recipient p95 | 1.12 ms | 1.2 ms | Effectively flat and below threshold |
| Sender-to-recipient p99 | 2.39 ms | 1.7 ms | -28.9% |
| Average server CPU | 19.7% | 10.7% | -45.5% |
| Delivery defects | 0 | 0 | No change |

Lower delivery volume is the intended result of sending fewer redundant position snapshots; it is not lost throughput. Remote rendering interpolates between the retained snapshots. At 240 clients across 16 rooms, the optimized run passed with 60.7% peak server CPU, while the comparable 60 Hz run failed its peak-CPU threshold at 118.5%.

## Measurement integrity

The harness records sequence numbers per sender and recipient, permitting explicit detection of missing and duplicate fan-out. It also records unexpected disconnects, generator emission attainment, separate server/generator CPU, RSS/heap, event-loop delay, and bounded-memory latency histograms.

Two preliminary attempts were excluded:

1. A 15-client aggregation attempt overflowed the harness call stack and produced no artifact. Aggregation was changed to an iterative implementation before the three official repetitions.
2. An initial ten-minute soak timed out while the child process flushed telemetry and produced no artifact. IPC finalization was fixed before the recorded passing soak.

These were harness failures, not application-capacity failures, and neither is included in the reported performance results.

## Interpretation limits

The results support claims about the isolated Socket.IO movement protocol and its local resource profile. They do not establish:

- production EC2 capacity or internet latency;
- end-to-end user capacity;
- browser rendering smoothness or frame rate;
- LiveKit/WebRTC media quality or SFU capacity;
- PostgreSQL or Next.js request performance;
- multi-instance Socket.IO behavior;
- the absence of memory leaks; or
- performance under mobile hardware, loss, jitter, or geographically distributed clients.

The tests are focused rather than exhaustive. Broad API-route coverage, browser end-to-end tests, media-quality benchmarks, staging load tests, coverage thresholds, and deployment smoke tests remain future work.

## Reproducibility

The private repository contains the executable tests, harness, predeclared thresholds, runbook, artifact manifest, and raw JSON. Review access can be provided for interview or formal technical evaluation.

Primary commands:

```bash
npm ci
npm run typecheck
npm test
npm run build
npm run benchmark:socket -- --clients 15 --rooms 1 --duration 15 --warmup 2 --drain 3 --hz 20
```

The implementation, tests, harness, and recorded artifacts were committed in private implementation commit `caf2088`; the final documentation reconciliation is represented by private commit `872b6e7`.
