# Fragility Scoring

The **Architecture Fragility Score** is a composite metric (0–100) that quantifies how susceptible a distributed architecture is to cascading failure. The score is computed identically across the web UI, CLI, and API to ensure consistency.

## Formula

```
Fragility = Cascade Susceptibility × 0.30
           + Partition Sensitivity  × 0.25
           + Quorum Fragility       × 0.20
           + Retry Storm Potential   × 0.15
           + SPOF Penalty
```

Where SPOF Penalty = `min(spof_count × 3, 15)`.

The raw value is clamped to `[0, 100]`.

## Sub-Score Computation

### Cascade Susceptibility (30%)

Measures how likely a single node failure propagates through the graph.

```
cascade = clamp(fanOutIndex × 12 + dependencyDensity × 40 + sccCyclePenalty)
```

Where:
- **Fan-Out Index** = max outgoing edges from any single node / total nodes
- **Dependency Density** = total edges / (nodes × (nodes - 1))
- **SCC Cycle Penalty** = number of strongly connected components with > 1 node × 8

SCC detection uses Tarjan's algorithm to identify circular dependency chains.

### Partition Sensitivity (25%)

Measures vulnerability to network partitions based on data plane configuration.

```
partition = avg(dataPlaneRisk(node)) for all nodes with dataPlaneConfig
```

Where `dataPlaneRisk` computes risk from:
- Consistency model (linearizable = highest risk under partition)
- Replication strategy (sync-all = highest latency risk)
- Sharding strategy (directory = highest coordination overhead)
- Transaction model (2PC = highest partition sensitivity)

If no nodes have data plane configs, falls back to topological estimate.

### Quorum Fragility (20%)

Measures risk from insufficient replication for consensus.

```
quorum = (nodes with replicaCount < 3) / total_nodes × 100
```

Nodes with fewer than 3 replicas cannot form a proper quorum, making them vulnerable to split-brain scenarios.

### Retry Storm Potential (15%)

Measures the risk of retry amplification cascades.

```
retry = clamp(retryAmplificationRisk × 15)
```

Where `retryAmplificationRisk` is computed from the product of retry policies across all edges, normalized by edge count.

### SPOF Penalty (+3 per node, max 15)

Identifies single points of failure via BFS disconnection analysis:

1. For each node, temporarily remove it from the graph
2. Run BFS from all remaining nodes
3. If removing the node disconnects the graph, it's an SPOF

Each SPOF adds 3 points to the fragility score, capped at 15.

## Risk Thresholds

| Score | Classification | Color | Interpretation |
|---|---|---|---|
| 0–39 | Low | 🟢 Green | Structural redundancy present |
| 40–69 | Moderate | 🟡 Yellow | Review SPOFs and retry policies |
| 70–100 | High | 🔴 Red | Cascading failure likely under stress |

## Fragility Trend

Fragility scores are tracked over time as **snapshots**. Each snapshot records:

- Timestamp
- Overall fragility score
- Topology contribution
- Mechanics contribution
- Data plane contribution
- Node and edge counts

Trends are visualized in the Architecture Builder and accessible via the CLI:

```bash
oex fragility trend <architecture-id> --limit 20
oex fragility trend <architecture-id> --output trend.csv --format csv
```

## Implementation Locations

The fragility formula is implemented in two locations that must stay synchronized:

1. **Frontend**: `src/utils/graphAnalysis.ts` → `computeStructuralRisk()`
2. **Backend API**: `supabase/functions/data-api/index.ts` → `computeStructuralAnalysis()`

Any modification to the formula must be applied to both locations to maintain CLI/UI consistency.
