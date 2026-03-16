# Methodology & Formulas

This document details the mathematical foundations behind OrchEngineX's structural analysis engine.

## Structural Metrics

### Fan-Out Index

Measures the maximum outgoing connectivity from any single node, normalized by graph size.

```
fanOutIndex = max(outDegree(v)) / |V|    for all v ∈ V
```

High fan-out indicates potential blast radius — a single node failure affects many downstream consumers.

### Dependency Density

Measures how interconnected the graph is relative to its maximum possible connectivity.

```
density = |E| / (|V| × (|V| - 1))
```

For directed graphs, maximum edges = `n × (n-1)`. Higher density means more failure propagation paths.

### Critical Path Depth

The longest shortest path in the graph, computed via BFS from all nodes.

```
criticalPathDepth = max(shortestPath(u, v))    for all u, v ∈ V
```

Deeper critical paths mean longer cascade chains and higher tail latency.

### Spectral Radius Approximation

Approximates the largest eigenvalue of the adjacency matrix using power iteration:

```
spectralRadius ≈ ||A^k × x|| / ||A^(k-1) × x||    as k → ∞
```

The spectral radius indicates the graph's amplification potential — values > 1 suggest instability under perturbation.

### Retry Amplification Risk

Measures the multiplicative retry pressure across all edges:

```
retryAmpRisk = Π(1 + retryPolicy(e) × packetLoss(e))    for all e ∈ E
```

Normalized to [0, 100]. Values above 50 indicate significant retry storm potential.

## SPOF Detection

Single Points of Failure are identified via BFS disconnection analysis:

```
For each node v ∈ V:
    G' = G \ {v}                     // Remove v and its edges
    components = BFS(G')              // Count connected components
    if components > 1:
        v is an SPOF
```

This is equivalent to finding articulation points in the underlying undirected graph, adapted for directed graphs by checking reachability.

## Strongly Connected Components

Tarjan's algorithm identifies cycles in the dependency graph:

```
function tarjan(v):
    v.index = v.lowlink = index++
    stack.push(v)
    for each (v, w) ∈ E:
        if w.index undefined:
            tarjan(w)
            v.lowlink = min(v.lowlink, w.lowlink)
        elif w on stack:
            v.lowlink = min(v.lowlink, w.index)
    if v.lowlink == v.index:
        emit SCC from stack
```

SCCs with > 1 node indicate circular dependencies that amplify cascading failures.

## Cascade Susceptibility

```
cascadeSusceptibility = clamp(
    fanOutIndex × 12 +
    dependencyDensity × 40 +
    sccCount × 8
, 0, 100)
```

Where `sccCount` is the number of strongly connected components with > 1 node.

## Partition Sensitivity

Computed from data plane configurations:

```
partitionSensitivity = avg(
    consistencyRisk(c) × 0.35 +
    replicationRisk(r) × 0.30 +
    shardingRisk(s) × 0.20 +
    transactionRisk(t) × 0.15
)    for all nodes with dataPlaneConfig
```

Risk scores per option:

| Config | Option | Risk |
|---|---|---|
| Consistency | linearizable | 85 |
| Consistency | sequential | 60 |
| Consistency | causal | 35 |
| Consistency | eventual | 10 |
| Replication | sync-all | 80 |
| Replication | sync-quorum | 50 |
| Replication | async-primary | 25 |
| Replication | async-mesh | 15 |
| Sharding | directory | 70 |
| Sharding | range | 45 |
| Sharding | hash | 20 |
| Sharding | none | 5 |
| Transaction | 2pc | 75 |
| Transaction | saga | 40 |
| Transaction | tcc | 55 |
| Transaction | none | 5 |

## Quorum Fragility

```
quorumFragility = (nodesWithReplicaCount < 3) / |V| × 100
```

The threshold of 3 is based on the minimum quorum requirement (⌊n/2⌋ + 1 = 2 for n=3).

## Composite Fragility Score

```
fragility = clamp(
    cascadeSusceptibility × 0.30 +
    partitionSensitivity × 0.25 +
    quorumFragility × 0.20 +
    retryStormPotential × 0.15 +
    min(spofCount × 3, 15)
, 0, 100)
```

## Node Impact Simulation

For a removed node `v`:

```
availabilityDrop = (1 - reachableNodes(G \ {v}) / |V|) × 100

latencyShift = Σ(latencyBaseline(v) × edgeWeight(e))
               for all edges e incident to v

retryAmpDelta = Π(1 + retryPolicy(e))
                for all edges e on paths through v

partitionRiskDelta = partitionSensitivity(G \ {v}) - partitionSensitivity(G)
```

## References

- Tarjan, R. E. (1972). "Depth-first search and linear graph algorithms"
- Newman, M. E. J. (2010). "Networks: An Introduction"
- Barabási, A-L. (2016). "Network Science"
- Veeraraghavan, K. et al. (2016). "Kraken: Leveraging Live Traffic Tests to Identify and Resolve Resource Utilization Bottlenecks in Large Scale Web Services"
