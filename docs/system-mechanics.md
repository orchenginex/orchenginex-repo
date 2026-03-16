# System Mechanics

The System Mechanics Simulator provides six composable engines that model different aspects of distributed system behavior. Each engine produces a composite risk score that feeds into the architecture's overall fragility assessment.

## Engines

### 1. Compute Engine

Models CPU/memory pressure and processing capacity.

**Parameters:**
- Thread pool size
- CPU utilization threshold
- Memory pressure level
- GC pause frequency

**Outputs:**
- Thread pool exhaustion probability
- GC amplification risk
- Processing latency under load

### 2. Consistency Engine

Models consistency guarantees and their operational costs.

**Parameters:**
- Consistency level (linearizable, sequential, causal, eventual)
- Read/write ratio
- Conflict resolution strategy
- Staleness tolerance

**Outputs:**
- Read/write conflict rate
- Consistency violation probability
- Coordination latency overhead

### 3. Replication Engine

Models data replication mechanics and failure modes.

**Parameters:**
- Replication strategy (sync-all, sync-quorum, async-primary, async-mesh)
- Replica count
- Network latency between replicas
- Failover timeout

**Outputs:**
- Replication lag distribution
- Split-brain probability
- Failover time estimate
- Data loss window

### 4. Transaction Engine

Models distributed transaction coordination costs.

**Parameters:**
- Transaction model (2PC, Saga, TCC, none)
- Lock granularity
- Transaction duration
- Concurrency level

**Outputs:**
- Lock contention rate
- Deadlock probability
- Coordination overhead (ms)
- Throughput impact

### 5. Distribution Engine

Models data distribution and sharding mechanics.

**Parameters:**
- Sharding strategy (none, hash, range, directory)
- Shard count
- Key distribution uniformity
- Rebalance frequency

**Outputs:**
- Hotspot probability
- Cross-shard query cost
- Rebalance duration
- Capacity utilization skew

### 6. Flow Control Engine

Models backpressure propagation and admission control.

**Parameters:**
- Buffer size
- Backpressure threshold
- Admission control policy
- Load shedding strategy

**Outputs:**
- Buffer saturation probability
- Backpressure propagation depth
- Load shedding rate
- Effective throughput under pressure

## Architecture Integration

### Derive from Architecture

The system can automatically derive mechanics configurations from architecture nodes:

```typescript
import { deriveConfigsFromArchitecture } from '@/components/mechanics/deriveFromArchitecture';

const configs = deriveConfigsFromArchitecture(nodes, edges);
// Returns default mechanics parameters based on node types and topology
```

### Apply to Architecture

Mechanics results can be applied back to architecture nodes for bidirectional sync:

```typescript
import { reverseApplyToArchitecture } from '@/components/mechanics/reverseApplyToArchitecture';

const updatedNodes = reverseApplyToArchitecture(nodes, mechanicsResults);
// Updates node failure probabilities and resource capacities
```

## Node Health Scoring

Each node receives a health score (0–100) based on:

| Factor | Weight | Source |
|---|---|---|
| Failure probability | 30% | Node configuration |
| Resource capacity | 25% | Node configuration |
| Latency baseline | 20% | Node + edge configuration |
| Node type risk | 15% | Type catalog defaults |
| Replica count | 10% | Quorum assessment |

### Health Classifications

| Score | Classification | Visual |
|---|---|---|
| 80–100 | Healthy | 🟢 Green bar |
| 50–79 | Degraded | 🟡 Yellow bar |
| 0–49 | Critical | 🔴 Red bar |

## Composite Risk

The mechanics composite risk feeds into the overall fragility score as the "mechanics contribution":

```
mechanicsRisk = weighted_avg(
  computeRisk × 0.20,
  consistencyRisk × 0.20,
  replicationRisk × 0.20,
  transactionRisk × 0.15,
  distributionRisk × 0.15,
  flowControlRisk × 0.10
)
```
