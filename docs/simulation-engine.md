# Simulation Engine

OrchEngineX provides multiple simulation capabilities for analyzing distributed system behavior under failure conditions.

## Node Removal Simulation

Simulates the impact of removing one or more nodes from the architecture graph.

### Metrics

| Metric | Description |
|---|---|
| **Availability Drop** | Percentage of nodes that become unreachable after removal |
| **Latency Shift** | Propagated latency increase through dependent paths |
| **Retry Amplification Delta** | Multiplier effect from retry policies on failed paths |
| **Partition Risk Delta** | Change in network partition sensitivity |

### Algorithm

1. Remove selected node(s) from the graph
2. Run BFS from all remaining nodes to compute reachability
3. Calculate availability as `reachable_nodes / total_nodes`
4. Traverse affected edges to compute latency propagation
5. Aggregate retry policies along failed paths for amplification estimate
6. Assess partition risk based on connectivity changes

## Cascade Propagation

Models how failure spreads through the graph using weighted BFS:

1. Start from failed node(s)
2. For each downstream neighbor:
   - Compute failure probability based on edge weight
   - Apply retry amplification at each hop
   - Check timeout threshold enforcement
3. Mark nodes as failed if accumulated failure probability exceeds threshold
4. Continue until no new nodes fail (steady state)

### Parameters

| Parameter | Description | Range |
|---|---|---|
| `failureNode` | Starting node(s) for failure injection | Node ID(s) |
| `latency` | Injected latency at failure point | ≥ 0 ms |
| `packetLoss` | Injected packet loss | 0–100% |
| `retries` | Override retry count | ≥ 0 |

## Multi-Node Failure

The simulator supports selecting multiple nodes for simultaneous failure, enabling analysis of:

- Correlated failures (e.g., all instances in one availability zone)
- Dependent failures (e.g., database + its replicas)
- Cascading failure chains

## Growth Mode Analysis

Predicts the efficiency of horizontal scaling by analyzing:

### Metrics

| Metric | Description |
|---|---|
| **Load Distribution** | Per-replica load and utilization percentage |
| **Retry Propagation** | Fan-out of retries under scaled topology |
| **Coordination Overhead** | Consensus latency for stateful nodes |
| **Mesh Hop Latency** | Additional latency from mesh routing at scale |
| **Scaling Efficiency** | Overall effectiveness score (0–100) |

### Computation

For each node with increased replica count:

```
scalingEfficiency = (capacityGain - coordinationCost - retryOverhead) / capacityGain × 100
```

Where:
- `capacityGain` = reduction in per-replica load
- `coordinationCost` = consensus latency × log2(replicaCount) for stateful nodes
- `retryOverhead` = additional retry fan-out from new replicas

## API

### REST Endpoint

```
POST /simulate
Content-Type: application/json
Authorization: Bearer <token>

{
  "architecture": { ... },        // or "architecture_id": "<uuid>"
  "failure_node": "db-primary",
  "latency": 150,
  "packet_loss": 5,
  "retries": 3
}
```

### CLI

```bash
oex simulate --arch <id> --failure-node db-primary --latency 150 --packet-loss 5 --retries 3
```

### Rate Limit

- 30 requests per minute per IP
- 429 response includes `X-RateLimit-Reset` header
