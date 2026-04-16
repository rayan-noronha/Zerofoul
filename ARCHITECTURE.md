# ZeroFoul — Architecture

## System Context (C4 Level 1)

This diagram describes how ZeroFoul fits into the broader ecosystem.

```
┌─────────────────────────────────────────────────────────────────┐
│                        EXTERNAL ACTORS                          │
│                                                                 │
│   [Bettor]          [Gambling Operator]          [Regulator]    │
│      │                      │                        │          │
│      │ places bet            │ integrates API         │ audits   │
│      ▼                      ▼                        ▼          │
└──────────────────────────────────────────────────────────────── ┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                         ZEROFOUL                                │
│                                                                 │
│        Verifiable Match Integrity Infrastructure                │
│                                                                 │
│  - Generates and commits match configs on-chain                 │
│  - Runs simulations inside a ZK virtual machine                 │
│  - Produces cryptographic proofs of correct execution           │
│  - Exposes verification API for operators and regulators        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      EXTERNAL SYSTEMS                           │
│                                                                 │
│   [Polygon Blockchain]     [Risc Zero zkVM]    [PostgreSQL]     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Container Diagram (C4 Level 2)

This diagram describes the major containers (applications and data stores) inside ZeroFoul.

```
┌─────────────────────────────────────────────────────────────────┐
│                          ZEROFOUL                               │
│                                                                 │
│  ┌─────────────────┐        ┌─────────────────┐                │
│  │   Sim Engine    │        │   Data Pipeline │                │
│  │   (Python)      │──────▶ │   (Airflow)     │                │
│  │                 │        │                 │                │
│  │ Generates match │        │ Orchestrates    │                │
│  │ events from a   │        │ config → hash   │                │
│  │ seeded RNG      │        │ → commit flow   │                │
│  └─────────────────┘        └────────┬────────┘                │
│                                      │                         │
│  ┌─────────────────┐                 │                         │
│  │  Proof Layer    │                 ▼                         │
│  │  (Risc Zero)    │        ┌─────────────────┐                │
│  │                 │        │   PostgreSQL     │                │
│  │ Wraps sim in    │        │                 │                │
│  │ zkVM, generates │        │ Stores match    │                │
│  │ ZK proof of     │        │ configs, events │                │
│  │ correct run     │        │ proofs, lineage │                │
│  └────────┬────────┘        └─────────────────┘                │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐        ┌─────────────────┐                │
│  │  Smart Contract │        │   FastAPI        │                │
│  │  (Solidity)     │        │                 │                │
│  │                 │        │ Exposes match    │                │
│  │ Stores config   │        │ config, proof    │                │
│  │ hash pre-match  │        │ and verification │                │
│  │ Verifies proof  │        │ endpoints        │                │
│  │ post-match      │        │                 │                │
│  └─────────────────┘        └─────────────────┘                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Data Flow — Match Lifecycle

```
PRE-MATCH
─────────
Config generated (teams, seed, difficulty, timestamp)
    │
    ▼
Config serialised → SHA256 hash produced
    │
    ▼
Hash committed to Polygon smart contract (on-chain, timestamped)
    │
    ▼
Betting opens — config is now immutable

EXECUTION
─────────
Match simulation runs inside Risc Zero zkVM
    │
    ▼
Match events captured (goals, cards, possession, final score)
    │
    ▼
ZK proof generated — proves simulation ran correctly on committed config
    │
    ▼
Outcome hash produced from match events

POST-MATCH
──────────
Outcome hash + ZK proof submitted to smart contract
    │
    ▼
Smart contract verifies proof on-chain
    │
    ▼
Outcome committed to blockchain — permanently verifiable
    │
    ▼
FastAPI exposes result — operator settles bets
```

---

## Layer Responsibilities

| Layer | Component | Responsibility |
|---|---|---|
| Layer 1 | Data Pipeline (Airflow) | Config generation, hashing, lineage tracking |
| Layer 1 | PostgreSQL | Persistent storage of configs, events, proofs |
| Layer 2 | Simulation Engine (Python) | Deterministic match simulation using seeded RNG |
| Layer 3 | Smart Contract (Solidity) | On-chain commitment and proof verification |
| Layer 3 | Polygon | Public, permanent, tamper-proof ledger |
| Layer 4 | Proof Layer (Risc Zero) | ZK proof generation wrapping the simulation |
| API | FastAPI | External interface for operators and regulators |
| Observability | Prometheus + Grafana | Pipeline health, proof generation metrics, alerts |

---

## Key Design Decisions

Decision records for each of the choices below are in the /docs folder.

- Why we build our own simulation engine rather than use EA FC — ADR-001
- Why we chose Polygon over Ethereum mainnet — ADR-002
- Why we chose Risc Zero over EZKL for ZK proofs — ADR-003

---

## Observability

Every layer is instrumented. The following metrics are tracked:

- Match config generation latency
- Config hash commitment time (on-chain)
- Simulation execution duration
- ZK proof generation duration and success rate
- Smart contract verification success rate
- API response times and error rates
- Pipeline DAG run status (Airflow)

Dashboards are defined in /observability/grafana/
Prometheus config is defined in /observability/prometheus/
