# Data Plane Simulator

The Data Plane Simulator models the impact of data layer configuration choices on system behavior, availability, and performance.

## Configuration Axes

### Consistency

| Level | Description | Risk Profile |
|---|---|---|
| `linearizable` | Strongest guarantee — all reads see latest write | Highest latency, lowest partition tolerance |
| `sequential` | Operations appear in same order to all observers | High coordination cost |
| `causal` | Causally related operations ordered | Moderate overhead |
| `eventual` | No ordering guarantee — convergence over time | Lowest latency, highest availability |

### Replication

| Strategy | Description | Risk Profile |
|---|---|---|
| `sync-all` | All replicas must acknowledge writes | Highest durability, lowest availability |
| `sync-quorum` | Majority of replicas must acknowledge | Balanced durability and availability |
| `async-primary` | Primary acknowledges, replicas lag | Low latency, data loss window |
| `async-mesh` | Peer-to-peer async replication | Highest availability, eventual consistency |

### Sharding

| Strategy | Description | Risk Profile |
|---|---|---|
| `none` | Single partition | No cross-shard overhead, limited scale |
| `hash` | Hash-based partitioning | Even distribution, no range queries |
| `range` | Range-based partitioning | Supports range queries, hotspot risk |
| `directory` | Lookup-based partitioning | Flexible, highest coordination overhead |

### Transaction

| Model | Description | Risk Profile |
|---|---|---|
| `2pc` | Two-phase commit | Strong consistency, blocking risk |
| `saga` | Compensating transactions | Eventually consistent, complex rollback |
| `tcc` | Try-Confirm-Cancel | Reservation-based, intermediate state exposure |
| `none` | No distributed transactions | Lowest overhead, no cross-service atomicity |

## Impact Metrics

The simulator produces four impact dimensions:

### Latency Impact
Additional latency introduced by the data plane configuration:
- Consistency coordination latency
- Replication acknowledgment latency
- Cross-shard routing latency
- Transaction coordination latency

### Availability Impact
Availability reduction from configuration choices:
- Sync replication reduces availability (requires more nodes up)
- Strong consistency reduces availability during partitions
- 2PC introduces blocking failure modes

### Partition Tolerance
Behavior during network partitions:
- Linearizable + sync-all = unavailable during partitions
- Eventual + async-mesh = available but inconsistent during partitions

### Throughput Impact
Write and read throughput changes:
- Write amplification from replication
- Read distribution from replicas
- Cross-shard query overhead
- Lock contention from transactions

## Usage

### Web UI

1. Open the **Architecture Builder**
2. Select a data-layer node (Database, Replica Set, Distributed KV, etc.)
3. Configure data plane settings in the node editor
4. View impact metrics in the **Data Plane** panel

### CLI

```bash
# Included in architecture inspect output
oex arch inspect <architecture-id>

# Full analysis via simulation
oex simulate --arch <id> --failure-node db-primary
```

### JSON Configuration

```json
{
  "id": "db-primary",
  "type": "database",
  "label": "PostgreSQL Primary",
  "dataPlaneConfig": {
    "consistency": "linearizable",
    "replication": "sync-quorum",
    "sharding": "hash",
    "transaction": "2pc"
  }
}
```

## Integration with Fragility Score

Data plane configuration feeds into the **Partition Sensitivity** sub-score (25% of total fragility), representing the architecture's vulnerability to network partitions based on its data coordination strategy.
