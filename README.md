# BLACKBOX Economy Engine

> A deterministic, replayable economic simulation where autonomous businesses set prices, manage supply chains, take on debt, and survive or go bankrupt. Trigger shocks, clone worlds, run A/B and Monte Carlo experiments, and trace any failure back to its root cause.

![Status](https://img.shields.io/badge/status-in%20development-amber)
![Java](https://img.shields.io/badge/Java-21-blue)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3-green)
![React](https://img.shields.io/badge/React-TypeScript-61dafb)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

> **Status:** Under active development as a 30-day build. The roadmap below tracks what is finished. Nothing is marked done until it works and its tests pass.

---

## Table of Contents

1. [What is BLACKBOX?](#what-is-blackbox)
2. [Design Principles](#design-principles)
3. [Features](#features)
4. [Architecture](#architecture)
5. [Tick Pipeline](#tick-pipeline)
6. [Algorithms and Why They Are Used](#algorithms-and-why-they-are-used)
7. [Tech Stack](#tech-stack)
8. [Repository Structure](#repository-structure)
9. [Getting Started](#getting-started)
10. [30-Day Roadmap](#30-day-roadmap)
11. [Testing and Quality](#testing-and-quality)
12. [Limitations](#limitations)
13. [License](#license)

---

## What is BLACKBOX?

BLACKBOX is a persistent artificial economy. Businesses, consumers, suppliers, and industries interact through markets, supply chains, and economic events. You can:

- Create simulated economic worlds and watch them run live.
- Inspect any business: its P&L, inventory, debt, and the reasons behind its decisions.
- Trigger shocks (demand crash, supplier failure, cost spike, interest-rate change) and watch them propagate.
- Clone a world at any point, change variables, and compare outcomes side by side.
- Run Monte Carlo experiments to get survival probabilities instead of single outcomes.
- Ask an AI analyst questions that are answered from real simulation data.

The simulation engine is the star. ML and AI are supporting layers that read state and never change it.

**This is a simplified, documented model. It is not a prediction of any real national economy.**

---

## Design Principles

1. **Deterministic.** The same seed and config always produce the same event-log hash.
2. **Money is conserved.** Every transfer is double-entry. Total money equals initial money plus logged injections, checked every tick.
3. **Agents propose, the engine decides.** Agents emit intents. The engine validates them against cash, stock, and capacity.
4. **Everything is traceable.** Every state change is an event with a `causeId`, which enables replay and causal tracing.
5. **Engine independent of everything else.** `engine-core` is pure Java with no Spring dependency.
6. **LLMs never mutate state.** The AI analyst has read-only tools.
7. **Money is `long` cents.** No floating-point currency.

---

## Features

| Feature | Description |
|---|---|
| Live simulation | Play, pause, step, and speed controls with a WebSocket tick stream |
| Autonomous businesses | Utility-scored policies for pricing, inventory, and cash management |
| Supply-chain graph | Interactive graph sized by systemic importance, with failure highlighting |
| Economic shocks | Scheduled or manual events with downstream propagation |
| Bankruptcy and liquidation | Insolvency rules with recorded losses |
| Causal tracing | "Why did this business fail?" answered as an event chain |
| Scenario cloning and A/B | Fork from a snapshot, change parameters, compare branches |
| Monte Carlo | Many seeds in parallel, with survival and percentile reports |
| Replay | Scrub to any tick from snapshots and the event log |
| ML risk prediction | Probability a business goes bankrupt within N ticks |
| AI analyst | Read-only tool-calling over analytics, citing tick and entity IDs |

---

## Architecture

A modular monolith. No Kafka, Kubernetes, or microservices unless the workload justifies it.

```
┌─────────────┐   REST / WebSocket   ┌──────────────────────────────┐
│  Frontend   │ ───────────────────► │  Spring Boot app             │
│ React + TS  │ ◄─────────────────── │  world · scenarios · API     │
└─────────────┘                      │  analytics · prediction · ai │
                                     └──────┬───────────────┬───────┘
                                            │               │
                              ┌─────────────▼───┐     ┌─────▼────────┐
                              │  engine-core    │     │ ML service   │
                              │  (pure Java)    │     │ FastAPI      │
                              └─────────────────┘     └──────────────┘
                                            │
                                     ┌──────▼──────┐
                                     │ PostgreSQL  │
                                     └─────────────┘
```

---

## Tick Pipeline

Each tick (one simulated day) runs these phases in a fixed order:

| # | Phase | What happens |
|---|---|---|
| 1 | Events | Scheduled shocks fire and apply effects |
| 2 | Perception | Agents observe prices, competitors, stock, cash |
| 3 | Decision | Agents score options and emit intents |
| 4 | Validation | Engine checks each intent; invalid ones are rejected and logged |
| 5 | Procurement | Supplier orders resolve; lead times create in-transit goods |
| 6 | Market clearing | Demand is allocated by price, reputation, and availability |
| 7 | Settlement | Ledger transfers for sales, wages, rent, interest |
| 8 | Accounting | P&L, cash flow, debt service, reputation update |
| 9 | Bankruptcy | Insolvency check and liquidation |
| 10 | Metrics | Aggregates, conservation check, periodic snapshot |

---

## Algorithms and Why They Are Used

| Algorithm | Purpose in BLACKBOX |
|---|---|
| Priority queue | Event scheduler and delivery arrivals, ordered by tick |
| Topological sort | Production order in the supply DAG; cycle detection on world validation |
| BFS | Shock propagation by hop distance |
| Dijkstra | Cheapest or fastest sourcing route (price plus lead time) |
| PageRank | Systemic importance of suppliers |
| Union-Find | Network fragmentation after bankruptcies |
| DFS | Causal-chain extraction from `causeId` links |
| Monte Carlo | Survival probability and outcome percentiles across seeds |

Time and space complexity, test cases, and benchmarks live in `docs/algorithms.md` and `docs/benchmarks.md` as each algorithm is built.

---

## Tech Stack

| Area | Technology |
|---|---|
| Engine | Java 21 (pure, no Spring) |
| Backend | Spring Boot, Spring Security, Spring Data JPA, Flyway |
| Database | PostgreSQL |
| Frontend | React, TypeScript, Vite, Tailwind CSS, Framer Motion, Zustand, TanStack Query, ECharts or Recharts, React Flow or Cytoscape.js |
| ML | Python, FastAPI, pandas, NumPy, scikit-learn, XGBoost (optional) |
| Infra | Docker, Docker Compose, GitHub Actions, Actuator and Prometheus |

---

## Repository Structure

```
blackbox-economy-engine/
├── backend/
│   ├── engine-core/     # Pure Java: model, ledger, tick, agents, markets,
│   │                    # supplychain, events, rng
│   ├── app/             # Spring Boot: world, simulation, scenarios,
│   │                    # analytics, prediction, ai, security, persistence
│   └── db/migrations/   # Flyway
├── ml-service/          # FastAPI: features, training, evaluation, /predict
├── frontend/            # React + TypeScript + Vite
├── docs/                # economic-model.md, algorithms.md, calibration.md,
│                        # benchmarks.md
├── docker-compose.yml
└── .github/workflows/
```

---

## Getting Started

> Commands below become valid as the matching roadmap days are completed.

```bash
# Clone
git clone https://github.com/Sidhkataria222-hub/blackbox-economy-engine.git
cd blackbox-economy-engine

# Configure environment
cp .env.example .env

# Run the full stack
docker compose up --build

# Run the engine headless (available after Day 7)
./gradlew :engine-core:run --args="run --config worlds/stable.json --ticks 1000"

# Run all tests
./gradlew test
```

Environment variables (see `.env.example`): database credentials, ML service URL, JWT secret, and the LLM API key. No secrets are committed to the repo.

---

## 30-Day Roadmap

Check items off only when the feature works and its tests pass.

### Week 1: Engine Core

- [ ] **Day 1:** Repo, Gradle multi-module, Docker Compose (Postgres), CI, economic model spec
- [ ] **Day 2:** Domain model, seeded RNG streams, world config and generator
- [ ] **Day 3:** Double-entry ledger with overdraft rejection and conservation invariant
- [ ] **Day 4:** Tick pipeline, event log with `causeId`, deterministic hash test
- [ ] **Day 5:** Consumer demand, price elasticity, market clearing
- [ ] **Day 6:** Inventory, procurement, supplier lead times, production
- [ ] **Day 7:** Expenses, debt, bankruptcy, liquidation, headless CLI

**Milestone:** 100 businesses run 1,000 ticks from the CLI, books balance, results deterministic.

### Week 2: Agents, Graph, Persistence

- [ ] **Day 8:** Agent framework: intents, utility scoring, strategy parameters
- [ ] **Day 9:** Pricing, inventory, and cash policies; competitor analysis
- [ ] **Day 10:** Calibration and three presets (`stable`, `competitive`, `fragile`)
- [ ] **Day 11:** Supply-chain graph, topological sort, Dijkstra sourcing, PageRank
- [ ] **Day 12:** Shock system with priority-queue scheduler and BFS propagation
- [ ] **Day 13:** Flyway schema, persistence of worlds, runs, events, snapshots
- [ ] **Day 14:** REST API with validation, error handling, integration tests

**Milestone:** The full engine is drivable through the API.

### Week 3: UI and Scenarios

- [ ] **Day 15:** Frontend shell, dark command-center theme, design tokens
- [ ] **Day 16:** WebSocket live stream and simulation controls
- [ ] **Day 17:** Core charts: output, prices, market share, bankruptcies
- [ ] **Day 18:** Business inspector with decision log and reasons
- [ ] **Day 19:** Supply-chain graph view with PageRank sizing and failure highlighting
- [ ] **Day 20:** Event timeline, shock panel, causal trace view
- [ ] **Day 21:** Scenario cloning and A/B comparison

**Milestone:** Clone a world, trigger a shock in one branch, compare the two.

### Week 4: Monte Carlo, ML, AI, Ship

- [ ] **Day 22:** Monte Carlo runner, survival and percentile report, benchmark
- [ ] **Day 23:** Replay scrubber from snapshots and event log
- [ ] **Day 24:** ML dataset export and feature engineering
- [ ] **Day 25:** Train, evaluate (baseline vs boosted), serve `/predict`, UI risk badges
- [ ] **Day 26:** AI analyst with read-only, whitelisted tools and cited answers
- [ ] **Day 27:** Spring Security, JWT, per-user worlds, rate limits
- [ ] **Day 28:** Logging, Prometheus metrics, determinism regression test in CI
- [ ] **Day 29:** Full Docker stack, README polish, algorithms and benchmark docs
- [ ] **Day 30:** Demo script, recording, limitations section, final cleanup

**Milestone:** A clean clone runs end to end from this README alone.

### Cut List (if behind schedule)

Cut in this order: Redis, DP optimization, Cytoscape polish, Arena, Reports, XGBoost (keep the logistic baseline), Replay UI.

**Never cut:** ledger invariant, determinism, event log with causes, scenario cloning, Monte Carlo.

---

## Testing and Quality

- **Determinism:** a golden-seed test fails CI if the event-log hash changes unintentionally.
- **Conservation:** the ledger invariant is asserted on every tick in tests.
- **Boundary:** a test verifies `engine-core` has no Spring imports.
- **Scenario cloning:** clones match the original exactly until the fork tick.
- **Monte Carlo:** results are identical regardless of thread count.
- **ML:** evaluated on held-out seeds, with the baseline model reported alongside.
- **AI:** a test proves the analyst cannot call any mutating endpoint.

---

## Limitations

- The economy is a simplified model and does not represent any real economy.
- ML risk predictions apply only to this simulated world and its parameters.
- Calibration is tuned for stability in the shipped presets; extreme custom configs may collapse or explode.
- Features not yet checked off in the roadmap are not implemented.

---

## License

MIT. See `LICENSE`.
