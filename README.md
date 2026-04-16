# ZeroFoul ⚽🔐

> Verifiable match integrity infrastructure for simulated sports.
> Cryptographic proof that every CPU vs CPU match ran fairly — committed on-chain before betting opens.

---

## What Is ZeroFoul?

ZeroFoul is a trustless match integrity layer for simulated sports betting platforms.

It solves one fundamental problem:

**How does a bettor know a simulated match wasn't rigged?**

Today they can't — they just trust the platform. ZeroFoul replaces that trust with math.

Every match goes through three cryptographic guarantees:

| Guarantee | What It Proves |
|---|---|
| Pre-Match Commitment | Match config was locked on-chain before any bet was placed |
| Execution Integrity | The simulation ran exactly on the committed conditions, unmodified |
| Outcome Verifiability | The result is provably derived from the execution — not from manipulation |

---

## How It Works

```
1. MATCH SCHEDULED
   → Config generated (teams, ratings, seed, difficulty)
   → Config hashed → committed to Polygon blockchain
   → Betting opens

2. BETTING PERIOD
   → Users place bets against a locked, immutable config
   → Nobody — including ZeroFoul — can change the outcome

3. MATCH EXECUTED
   → Simulation runs inside a ZK virtual machine (Risc Zero)
   → Full match state captured event by event
   → ZK proof generated proving correct execution

4. SETTLEMENT
   → Outcome hash + ZK proof submitted to smart contract
   → Contract verifies proof on-chain
   → Outcome is publicly verifiable forever

5. AUDIT TRAIL
   → Full lineage stored — config → execution → proof → outcome
   → Any regulator or auditor can independently verify any match, any time
```

---

## Architecture

```
┌─────────────────────────────────────────┐
│           LAYER 4 — PROOF LAYER         │
│     ZK Proof Generation & Verification  │
│              (Risc Zero)                │
├─────────────────────────────────────────┤
│         LAYER 3 — BLOCKCHAIN LAYER      │
│    Commitment Storage & Verification    │
│              (Polygon)                  │
├─────────────────────────────────────────┤
│         LAYER 2 — EXECUTION LAYER       │
│      Match Runner + State Capture       │
│         (Python Simulation Engine)      │
├─────────────────────────────────────────┤
│           LAYER 1 — DATA LAYER          │
│   Match Config, Ingestion & Lineage     │
│         (FastAPI + PostgreSQL)          │
└─────────────────────────────────────────┘
```

Full C4 architecture diagrams available in docs/ARCHITECTURE.md

---

## Tech Stack

| Layer | Technology |
|---|---|
| Simulation Engine | Python |
| ZK Proof Generation | Risc Zero |
| Blockchain | Polygon (EVM) |
| Smart Contracts | Solidity |
| Data Pipeline | Python + Airflow |
| API | FastAPI |
| Infrastructure | Terraform + Docker |
| Observability | Prometheus + Grafana |

---

## Repo Structure

```
zerofoul/
├── sim-engine/        # Football simulation engine (Python)
├── contracts/         # Smart contracts (Solidity)
├── proof-layer/       # ZK proof generation (Risc Zero / Rust)
├── data-pipeline/     # Match config pipeline and lineage
├── api/               # FastAPI endpoints
├── infra/             # Terraform IaC
├── observability/     # Grafana dashboards, Prometheus config
└── docs/
    ├── ARCHITECTURE.md
    ├── ADR-001-simulation-engine.md
    ├── ADR-002-blockchain-choice.md
    └── ADR-003-zk-framework.md
```

---

## Who Is This For?

ZeroFoul is B2B infrastructure. We are not a gambling platform.

Our customers are:

- E-sports betting platforms needing provably fair match outcomes
- Online gaming platforms needing verifiable RNG
- Regulators needing independent audit access to match integrity
- Fantasy sports platforms needing verifiable simulation outcomes

---

## Project Status

Phase 1 — Active Development

- [ ] Simulation engine (Python)
- [ ] Match config data pipeline
- [ ] Blockchain commitment layer (Polygon testnet)
- [ ] ZK proof generation (Risc Zero)
- [ ] Smart contract (Solidity)
- [ ] FastAPI layer
- [ ] Observability stack
- [ ] End-to-end integration

---

## Built By

**Rayan Noronha** — Data & AI Platform Engineer
Adelaide, SA, Australia

- linkedin.com/in/rayannoronha
- rippaa.com
- github.com/rayan-noronha

---

## License

MIT — open source from day one.
The simulation engine is publicly auditable by design. That is the point.
