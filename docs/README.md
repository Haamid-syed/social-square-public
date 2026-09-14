# Social Square engineering blueprint

This directory documents the current private Social Square implementation without publishing its application code.

## Suggested reading paths

- **Project overview:** [Main README](../README.md), [Architecture](ARCHITECTURE.md), [Verification and benchmarks](VERIFICATION_AND_BENCHMARKS.md), and [Current state](CURRENT_STATE.md).
- **Technical review:** [Component map](COMPONENT_MAP.md), [Real-time protocol](REALTIME_PROTOCOL.md), [Data and API](DATA_AUTH_API.md), [Design decisions](DESIGN_DECISIONS.md), and [Engineering change log](CHANGELOG.md).
- **Operations review:** [Deployment and operations](DEPLOYMENT_OPERATIONS.md) and [Full architecture map](FULL_ARCHITECTURE_MAP.md).

## Document index

| Reading order | Document | Focus |
|---:|---|---|
| 1 | [Architecture](ARCHITECTURE.md) | System context, containers, runtime boundaries, and end-to-end flows |
| 2 | [Component map](COMPONENT_MAP.md) | Important source files, responsibilities, and dependencies |
| 3 | [Real-time protocol](REALTIME_PROTOCOL.md) | Multiplayer events, room lifecycle, media plane, and failure behavior |
| 4 | [Data, auth, and API](DATA_AUTH_API.md) | Persistence, identity, cookies, protected routes, and HTTP endpoints |
| 5 | [Deployment and operations](DEPLOYMENT_OPERATIONS.md) | Image delivery, runtime topology, secrets, persistence, and observability |
| 6 | [Design decisions](DESIGN_DECISIONS.md) | Architectural choices and their tradeoffs |
| 7 | [Engineering change log](CHANGELOG.md) | Dated build, security, correctness, performance, verification, and cleanup changes |
| 8 | [Verification and benchmarks](VERIFICATION_AND_BENCHMARKS.md) | Automated tests, methodology, thresholds, measured results, and limitations |
| 9 | [Current state and roadmap](CURRENT_STATE.md) | Implemented, partial, modeled, and planned capabilities |
| Reference | [Full architecture map](FULL_ARCHITECTURE_MAP.md) | Compact, cross-cutting map of the complete system |

## Documentation conventions

- Disabled or unconnected behavior is labeled as partial, modeled, or planned.
- File names, responsibilities, events, and route behavior are documented; implementation bodies remain private.
- Credentials and account-specific infrastructure identifiers are excluded.
- Durable database models are distinguished from ephemeral live-room state.
- The documentation was verified on 15 September 2026 against private `main` commit `872b6e7`.

Return to the [project overview](../README.md).
