# BlackBox-economy-engine
BLACKBOX is a deterministic economic simulation engine where autonomous businesses set prices, manage supply chains, take on debt, and survive or go bankrupt. Trigger shocks, clone worlds, run A/B and Monte Carlo experiments, replay any run, and trace failures to their root causes. ML predicts business risk; an AI analyst explains real data.  


Think of BLACKBOX as five layers. Each layer only talks to the one next to it, so you can change one without breaking the others.

1. The Engine (the brain)

This is a pure Java module with no web or database code. Its only job is to take a world state and advance it by one tick, following fixed rules. It works like a game engine for economics: you give it a starting state and a seed, and it produces the next state.

Inside it are five ideas:

State: a snapshot of the world (businesses, consumers, suppliers, products, cash, stock).
Rules: how demand, pricing, production, and bankruptcy work.
Ledger: a double-entry record of every money movement, so money can never appear or vanish.
Event log: an append-only history of everything that happened, each entry pointing to its cause.
Seeded randomness: a controlled source of chance so the same seed always gives the same result.
2. The Agents (the decision-makers)

Businesses don’t change the world directly. Each tick, a business looks at what it can see (prices, rivals, its own cash and stock), scores its options, and submits intents such as “set price to X” or “order 100 units.”

The engine then acts as a referee. It checks each intent against reality (does the business have the cash? does the stock exist?) and either executes it or rejects it with a logged reason. This separation of proposing from deciding is what stops any agent, or any AI, from cheating the economy.

3. The Tick (how time works)

Time moves in discrete steps. Every tick runs the same phases in the same order:

Shocks fire.
Agents observe.
Agents decide.
The engine validates.
Suppliers deliver.
Consumers buy.
Money settles.
Books update.
Insolvent businesses fail.
Metrics are recorded.

A fixed order matters because it makes outcomes predictable and debuggable. If the order changed, the same seed could give different results.

4. The Application Layer (the wrapper)

Spring Boot wraps the engine and adds everything the engine deliberately doesn’t know about:

API: lets the frontend create worlds and control runs.
Persistence: saves worlds, events, and snapshots to PostgreSQL.
Scenarios: forks a world from a snapshot to create a branch.
Analytics: computes comparisons and Monte Carlo summaries.
Security: users, ownership, and rate limits.

Because the engine is separate, you can run thousands of copies in memory for Monte Carlo without touching the web layer at all.

5. The Intelligence Layer (the observers)

Two services read the simulation but never write to it:

ML service: learns from many past runs to estimate the chance a business goes bankrupt soon. It is only a prediction about this simulated world.
AI analyst: answers questions by calling read-only tools on real data and citing the tick and entity involved. It explains the numbers and never invents them.
6. The Frontend (the window)

The UI doesn’t hold any economic logic. It displays what the backend streams to it: live charts, the business inspector, the supply graph, the event timeline, and scenario comparisons.

How the pieces connect
User action → API → Engine advances world → Events + snapshots saved
                                          → Metrics streamed to UI
Saved data → ML trains / Analyst reads → insights shown in UI
The four ideas that hold it together
Idea	Why it matters
Determinism	Same seed gives the same result, which makes experiments trustworthy
Conservation	Total money is always accounted for, which keeps the economy honest
Traceability	Every change has a cause, so you can answer “why did this happen?”
Separation	The engine, agents, API, and AI each have one job, which keeps the project buildable in 30 days
Why this order of building

You build from the inside out: engine first, then agents, then persistence and API, then UI, then ML and AI. Each layer depends only on the ones before it, so at any point in the 30 days you have something that works, rather than a half-finished everything.
