# ADR-001 — Build a Custom Simulation Engine

## Status

Accepted

## Date

2026-04-16

## Context

ZeroFoul requires a football simulation engine to generate CPU vs CPU match outcomes. The most obvious option is to use an existing commercial game — EA FC (formerly FIFA) — running on PC in an automated mode. However, several alternatives exist:

- Option A — Use EA FC running in automated mode on PC
- Option B — Build a custom open-source football simulation engine in Python
- Option C — License an existing indie football simulation engine

The choice of simulation engine directly impacts the trust model, legal risk, and verifiability of the entire platform.

## Decision

We will build a custom, open-source football simulation engine in Python (Option B).

## Reasoning

### Against Option A — EA FC

EA FC is proprietary software owned by Electronic Arts. Using it as the foundation of a commercial gambling integrity product creates the following problems.

EA's End User License Agreement (EULA) explicitly prohibits commercial use of the game engine without a separate licensing agreement. Running automated matches for the purpose of gambling-adjacent products almost certainly violates these terms.

EA is known to aggressively enforce its IP through cease and desist orders. Building a business on their foundation introduces existential legal risk that cannot be mitigated without a direct commercial agreement with EA — which is unlikely to be granted to an early stage project.

Critically, a closed game engine cannot be independently audited. If ZeroFoul's core value proposition is verifiable fairness, using an unauditable black box simulation undermines the entire trust model.

### Against Option C — Indie Licensing

Licensing an existing indie engine introduces a dependency on a third party's codebase, release schedule, and business decisions. It also limits how deeply we can instrument and wrap the simulation for ZK proof generation. Risc Zero requires the simulation logic to run inside its zkVM — this is far easier to achieve with code we fully own and control.

### For Option B — Custom Engine

A custom Python simulation engine gives ZeroFoul full ownership of the most critical component in the trust chain.

The engine can be open-sourced from day one, making the simulation logic publicly auditable. Any bettor, regulator, or researcher can read exactly how match outcomes are generated. This is a feature, not a liability — it is the foundation of the trust model.

The engine can be designed from the ground up for determinism. Given the same random seed and the same team configurations, it will always produce the same match outcome. This determinism is a requirement for ZK proof generation.

The engine can be directly wrapped inside Risc Zero's zkVM without translation layers or compatibility shims. The simulation logic runs natively in the proving environment.

The simulation parameters — team ratings, form, home advantage, fatigue, match momentum — can be modelled transparently and tuned over time based on real football statistics.

## Consequences

### Positive

- Full legal ownership of the simulation layer
- Publicly auditable — open source from day one
- Designed for determinism, enabling clean ZK proof generation
- No dependency on EA's licensing decisions or EULA changes
- Can be instrumented at any level for observability and testing

### Negative

- Requires upfront engineering effort to build the engine
- Match outcomes will feel less realistic than a commercial game engine initially
- No visual broadcast layer — matches are data events, not rendered graphics
- Requires ongoing calibration to ensure team ratings reflect real-world performance

## Mitigations

The lack of visual output is acceptable at Phase 1. The broadcast layer (rendering match events as a visual experience) is a future concern and can be built on top of the simulation data layer independently.

Realism of outcomes will improve over time as team rating models are refined against real football statistics. Initial ratings will be seeded from publicly available football data sources.

## Review Date

Revisit this decision at the end of Phase 1 (Month 3) once the simulation engine is running and the ZK proof layer has been validated against it.
