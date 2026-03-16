<p align="center">
  <img src="public/favicon.png" alt="OrchEngineX" width="64" height="64" />
</p>

<h1 align="center">OrchEngineX</h1>

<p align="center">
  <strong>Structural Fragility Modeling for Distributed Systems</strong><br/>
  Model, simulate, and quantify cascading failure risk — before production.
</p>

<p align="center">
  <a href="https://www.orchenginex.com">Platform</a> ·
  <a href="https://www.orchenginex.com/docs">Documentation</a> ·
  <a href="https://www.orchenginex.com/publications">Research</a> ·
  <a href="https://www.orchenginex.com/simulations">Simulations</a> ·
  <a href="#cli">CLI</a>
</p>

---

## What is OrchEngineX?

OrchEngineX is a **Distributed Systems & Infrastructure Research Laboratory** focused on structural fragility modeling. It provides a simulation platform for engineers to model distributed architectures as directed weighted failure graphs (DWFGs) and quantify cascading failure risk through composable simulation engines.

**The core thesis:** Most production outages aren't caused by hardware failures — they're caused by structural coordination problems that are invisible to monitoring dashboards. OrchEngineX makes those structural risks visible and quantifiable before they reach production.

### Key Capabilities

| Capability | Description |
|---|---|
| **Architecture Builder** | Visual DWFG editor with 22 node types across 7 infrastructure layers |
| **Fragility Scoring** | Composite risk model (0–100) combining topology, mechanics, and data plane analysis |
| **Simulation Engine** | Node removal, cascade propagation, retry storm, and partition sensitivity modeling |
| **System Mechanics** | 6-engine analysis: compute, consistency, replication, transactions, distribution, flow control |
| **Data Plane Simulator** | Consistency/replication/sharding/transaction impact modeling |
| **Growth Mode Analysis** | Scaling efficiency prediction with coordination overhead estimation |
| **Failure Case Studies** | Reproducible research on pool saturation cascades, queue collapse, and more |
| **CLI Tooling** | Full command-line interface for automation, CI/CD integration, and batch analysis |

---

## Architecture Model

OrchEngineX models distributed systems as **Directed Weighted Failure Graphs (DWFGs)** — a graph-theoretic representation where:

- **Nodes** represent infrastructure components (API gateways, databases, caches, message brokers, etc.)
- **Edges** represent communication paths with latency, packet loss, retry, and timeout characteristics
- **Weights** encode failure probability, resource capacity, and retry amplification potential

### Node Types (22 types across 7 layers)

| Layer | Node Types |
|---|---|
| **Edge** | CDN, Edge Cache, Geo DNS / Traffic Router |
| **Ingress** | API Gateway, Load Balancer, Reverse Proxy, WAF |
| **Compute** | Stateless Service, Stateful Service, Background Worker, Batch Processor |
| **Data** | Database (Primary), Replica Set, Cache, Distributed KV Store, Search Index |
| **Messaging** | Message Broker, Stream Processor, Dead Letter Queue |
| **Mesh** | Service Mesh Proxy |
| **Egress** | External API, Payment Gateway, Third-Party Service |

### Edge Types

| Type | Description | Default Timeout |
|---|---|---|
| `sync` | Synchronous request/response | 3,000ms |
| `async` | Asynchronous event delivery | 30,000ms |
| `replication` | Data replication between stores | 60,000ms |
| `mesh-hop` | Service mesh sidecar routing | 5,000ms |

### System Constraints

- Maximum **40 nodes** per architecture
- Maximum **80 edges** per architecture
- No self-loops permitted
- Unique node and edge IDs enforced
- File size limit: **2MB** for JSON imports

---

## Fragility Score

The **Architecture Fragility Score** is a composite metric (0–100) that quantifies how susceptible a distributed architecture is to cascading failure. It is computed identically across the web UI, API, and CLI.

### Composition

```
Fragility = Cascade Susceptibility × 0.30    (topology)
           + Partition Sensitivity  × 0.25    (data plane)
           + Quorum Fragility       × 0.20    (quorum)
           + Retry Storm Potential   × 0.15    (mechanics)
           + SPOF Penalty                      (nodes × 3, max 15)
```

### Sub-Scores

| Factor | Weight | What It Measures |
|---|---|---|
| **Cascade Susceptibility** | 30% | Fan-out amplification × dependency density × SCC cycle penalty |
| **Partition Sensitivity** | 25% | Data plane contribution from consistency/replication/sharding/transaction configs |
| **Quorum Fragility** | 20% | Risk from nodes with replicaCount < 3 (quorum threshold) |
| **Retry Storm Potential** | 15% | Retry amplification risk from retry policies across all edges |
| **SPOF Penalty** | +3/node | Single points of failure detected via BFS disconnection analysis |

### Risk Thresholds

| Score | Classification | Interpretation |
|---|---|---|
| **0–39** | 🟢 Low | Architecture has structural redundancy and controlled retry geometry |
| **40–69** | 🟡 Moderate | Some structural risks present — review SPOFs and retry policies |
| **70–100** | 🔴 High | Architecture is structurally fragile — cascading failure likely under stress |

---

## Simulation Engine

### Node Removal Impact

Simulates the removal of one or more nodes and computes:

- **Availability Drop** — percentage of unreachable nodes post-failure
- **Latency Shift** — propagated latency increase through dependent paths
- **Retry Amplification Delta** — multiplier effect from retry policies on failed paths
- **Partition Risk Delta** — change in network partition sensitivity

### Cascade Propagation

Models how failure spreads through the graph using:

1. BFS traversal from failed node(s)
2. Weighted propagation based on edge failure probabilities
3. Retry amplification at each hop
4. Timeout threshold enforcement

### Growth Mode

Predicts the efficiency of horizontal scaling by analyzing:

- **Load distribution** per replica
- **Retry propagation** fan-out under scale
- **Coordination overhead** (consensus latency for stateful nodes)
- **Mesh hop latency** increase from additional replicas
- **Overall scaling efficiency** (0–100)

---

## System Mechanics

Six composable simulation engines model different aspects of distributed system behavior:

| Engine | What It Models |
|---|---|
| **Compute** | CPU/memory pressure, thread pool exhaustion, GC amplification |
| **Consistency** | Linearizable vs. eventual consistency trade-offs, read/write conflict rates |
| **Replication** | Sync vs. async replication lag, split-brain probability, failover timing |
| **Transaction** | 2PC vs. Saga coordination overhead, lock contention, deadlock probability |
| **Distribution** | Hash vs. range vs. directory sharding, hotspot probability, rebalance cost |
| **Flow Control** | Backpressure propagation, buffer saturation, admission control effectiveness |

Each engine produces a **composite risk score** that feeds into the architecture's overall fragility assessment.

---

## Data Plane Simulator

Models the impact of data layer configuration choices:

### Configuration Axes

| Axis | Options |
|---|---|
| **Consistency** | `linearizable`, `sequential`, `causal`, `eventual` |
| **Replication** | `sync-all`, `sync-quorum`, `async-primary`, `async-mesh` |
| **Sharding** | `none`, `hash`, `range`, `directory` |
| **Transaction** | `2pc`, `saga`, `tcc`, `none` |

### Impact Metrics

- **Latency Impact** — additional latency from coordination
- **Availability Impact** — availability reduction from consistency requirements
- **Partition Tolerance** — behavior during network splits
- **Throughput Impact** — write/read throughput changes

---

## CLI

The `oex` CLI provides full command-line access to OrchEngineX capabilities. Distributed as the `@orchenginex/cli` npm package.

### Installation

```bash
npm install -g @orchenginex/cli
```

### Authentication

```bash
# Browser-based OAuth (recommended)
oex login

# Token-based (CI/CD environments)
oex login --token <your-api-token>

# Verify authentication
oex whoami
```

### Commands

#### Architecture Management

```bash
# List all saved architectures
oex arch list

# Import architecture from JSON
oex arch import ./my-architecture.json --name "Production Topology"

# Inspect architecture with structural analysis
oex arch inspect <architecture-id>

# Delete an architecture
oex arch delete <architecture-id> --force
```

**`oex arch inspect` output:**

```
  Production Topology
  ─────────────────────────────────────
  Nodes:                 18
  Edges:                 24
  Density:               15.7%
  Fan-Out Index:         2.4
  Critical Path Depth:   6
  Spectral Radius:       3.1

  Fragility Score:       42/100
  Cascade Susceptibility:38
  Partition Sensitivity: 45
  Retry Amp Risk:        31

  SPOFs:                 db-primary, api-gateway
```

#### Fragility Analysis

```bash
# Get current fragility score
oex fragility score <architecture-id>

# View fragility trend (historical snapshots)
oex fragility trend <architecture-id> --limit 20

# Export trend data
oex fragility trend <architecture-id> --output trend.csv --format csv
```

#### Simulation

```bash
# Run failure simulation
oex simulate --failure-node api-gateway --latency 150 --packet-loss 5 --retries 3

# Run with saved architecture
oex simulate --arch <architecture-id> --failure-node db-primary
```

#### Experiments (Matrix Sweeps)

```bash
# Run parameter sweep experiment
oex experiments run --arch <architecture-id>

# List experiment runs
oex experiments list

# Generate experiment report
oex experiments report <experiment-id>
```

#### Visualization

```bash
# Generate architecture graph
oex visualize graph <architecture-id> --format svg --output arch.svg
oex visualize graph <architecture-id> --format png --output arch.png

# Generate fragility heatmap
oex visualize heatmap <architecture-id> --format svg

# Generate trend chart
oex visualize trend <architecture-id> --format svg
```

#### Data Export

```bash
# Export architecture data
oex export architecture <id> --format json
oex export architecture <id> --format yaml

# Export simulation results
oex export simulation <id> --format csv

# Export fragility snapshots
oex export fragility <id> --format csv
```

#### Project Scaffolding

```bash
# Initialize new project with starter template
oex init

# Validate architecture JSON offline
oex validate ./architecture.json

# Direct structural analysis
oex analyze structure ./architecture.json
```

### Rate Limits

| Endpoint | Limit |
|---|---|
| `/simulate` | 30 req/min |
| `/visualize` | 20 req/min |
| `/data-api` | 60 req/min |
| `/experiments` | 10 req/min |

### Configuration

- Project manifest: `.oex.yml`
- Credentials: `~/.oex/credentials.json` (0o600 permissions)
- API token env var: `OEX_API_TOKEN`

---

## Architecture JSON Schema

Architectures are defined as JSON files conforming to the `CustomArchitecture` interface:

```jsonc
{
  "name": "My Architecture",
  "nodes": [
    {
      "id": "gw-1",
      "type": "api-gateway",          // one of 22 node types
      "label": "API Gateway",
      "latencyBaseline": 5,            // ms
      "retryPolicy": 2,               // max retries
      "failureProbability": 1,         // 0-100%
      "resourceCapacity": 95,         // 0-100%
      "replicaCount": 2,              // optional, default 1
      "dataPlaneConfig": {             // optional
        "consistency": "eventual",
        "replication": "async-primary",
        "sharding": "hash",
        "transaction": "saga"
      }
    }
  ],
  "edges": [
    {
      "id": "e-1",
      "source": "gw-1",
      "target": "svc-orders",
      "type": "sync",                  // sync | async | replication | mesh-hop
      "latencyDistribution": 10,       // avg ms
      "packetLossProbability": 0.5,    // 0-100%
      "retryPolicy": 2,               // max retries
      "timeoutThreshold": 3000         // ms
    }
  ]
}
```

See [`examples/`](examples/) for complete importable architectures.

---

## Research & Publications

OrchEngineX publishes failure case studies with reproducible simulations:

| Publication | Key Finding |
|---|---|
| [The 4-Minute Queue Collapse](https://www.orchenginex.com/publications/queue-collapse-traffic-spike) | 10% traffic spike → total system collapse in 3:47 via queue depth amplification |
| [Pool Saturation Cascade](https://www.orchenginex.com/publications/pool-saturation-cascade-structural-fragility) | 2% latency spike → 20-service cascading failure via connection pool exhaustion |

Each publication includes interactive simulations, architecture graphs, and exportable diagrams.

---

## Project Structure

```
orchenginex/
├── public/                          # Static assets
│   ├── favicon.png                  # Circuit-style favicon
│   └── robots.txt
├── src/
│   ├── components/
│   │   ├── architecture/            # Architecture Builder & analysis
│   │   │   ├── ArchitectureBuilder.tsx
│   │   │   ├── ArchitectureGraphView.tsx
│   │   │   ├── ArchitectureMechanicsPanel.tsx
│   │   │   ├── FragilityTrendChart.tsx
│   │   │   ├── GrowthModePanel.tsx
│   │   │   ├── SimulationPanel.tsx
│   │   │   └── StructuralAnalysis.tsx
│   │   ├── mechanics/               # System mechanics engines
│   │   │   ├── computeMechanics.ts
│   │   │   ├── computeNodeHealth.ts
│   │   │   ├── deriveFromArchitecture.ts
│   │   │   └── MechanicsSimulator.tsx
│   │   ├── dataplane/               # Data plane simulator
│   │   │   ├── DataPlaneSimulator.tsx
│   │   │   └── computeImpact.ts
│   │   ├── pool-saturation/         # Pool saturation case study
│   │   ├── queue-collapse/          # Queue collapse case study
│   │   └── docs/                    # Documentation components
│   ├── types/
│   │   └── architecture.ts          # Core type definitions (DWFG)
│   ├── utils/
│   │   ├── graphAnalysis.ts         # Structural metrics & risk computation
│   │   ├── importArchitectureValidator.ts
│   │   ├── exportGraphImage.ts
│   │   └── exportGraphVideo.ts
│   ├── hooks/
│   │   ├── useArchitecturePersistence.ts
│   │   ├── useFragilityTracking.ts
│   │   └── useSavedSimulations.ts
│   ├── pages/
│   │   ├── Index.tsx                # Landing page
│   │   ├── Simulations.tsx          # Architecture Builder + Simulations
│   │   ├── Publications.tsx         # Research publications
│   │   ├── Docs.tsx                 # Technical documentation hub
│   │   ├── FailureAnalyses.tsx      # Failure case studies index
│   │   ├── Methodology.tsx          # Methodology & formulas
│   │   └── Performance.tsx          # Performance analysis
│   └── data/
│       └── publications.tsx         # Publication content & metadata
├── supabase/
│   └── functions/                   # Backend edge functions
│       ├── data-api/                # Architecture CRUD, fragility API
│       ├── simulate/                # Simulation engine API
│       ├── visualize/               # SVG/PNG/HTML generation
│       ├── experiments/             # Matrix sweep experiments
│       ├── cli-auth/                # CLI authentication flow
│       └── health-monitor/          # Health check endpoint
├── docs/                            # Extended documentation
│   ├── architecture-modeling.md
│   ├── fragility-scoring.md
│   ├── simulation-engine.md
│   ├── system-mechanics.md
│   ├── data-plane.md
│   ├── cli-reference.md
│   └── methodology.md
├── examples/                        # Importable architecture JSON files
│   ├── microservices-basic.json
│   ├── event-driven-pipeline.json
│   ├── multi-region-ha.json
│   └── full-stack-hft.json
├── CONTRIBUTING.md
├── CHANGELOG.md
├── LICENSE
└── README.md
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React 18, TypeScript, Vite |
| **Styling** | Tailwind CSS, shadcn/ui |
| **Charts** | Recharts |
| **Animations** | Framer Motion |
| **Backend** | Supabase (Edge Functions, PostgreSQL, Auth) |
| **CLI** | Node.js, Commander.js |
| **Deployment** | Vercel |

---

## Development

### Prerequisites

- Node.js 18+
- npm or bun

### Setup

```bash
# Clone the repository
git clone https://github.com/orchenginex/orchenginex.git
cd orchenginex

# Install dependencies
npm install

# Start development server
npm run dev
```

### Environment Variables

| Variable | Description |
|---|---|
| `VITE_SUPABASE_URL` | Backend API URL |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Public API key |

### Testing

```bash
# Run unit tests
npm test

# Run edge function tests
npx supabase functions test simulate
```

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on:

- Reporting bugs and requesting features
- Submitting pull requests
- Code style and architecture conventions
- Adding new node types or simulation engines

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

<p align="center">
  <strong>OrchEngineX</strong> — Structural Fragility Modeling for Distributed Systems<br/>
  <a href="https://www.orchenginex.com">www.orchenginex.com</a>
</p>
