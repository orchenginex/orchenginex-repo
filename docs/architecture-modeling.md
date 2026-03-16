# Architecture Modeling

OrchEngineX models distributed systems as **Directed Weighted Failure Graphs (DWFGs)** — a graph-theoretic abstraction where nodes represent infrastructure components and edges represent communication paths with failure characteristics.

## Core Concepts

### Directed Weighted Failure Graph (DWFG)

A DWFG `G = (V, E, W)` consists of:

- **V** — a set of typed infrastructure nodes
- **E** — a set of directed edges representing communication paths
- **W** — weight functions encoding latency, failure probability, retry policy, and timeout thresholds

This representation enables graph-theoretic analysis of structural risk properties that are invisible in traditional architecture diagrams.

### Node Model

Each node in the DWFG has the following properties:

| Property | Type | Range | Description |
|---|---|---|---|
| `id` | string | — | Unique identifier |
| `type` | NodeType | 22 types | Infrastructure component type |
| `label` | string | — | Human-readable name |
| `latencyBaseline` | number | ≥ 0 ms | Baseline processing latency |
| `retryPolicy` | number | ≥ 0 | Maximum retry attempts |
| `failureProbability` | number | 0–100% | Probability of failure per request |
| `resourceCapacity` | number | 0–100% | Available resource headroom |
| `replicaCount` | number | ≥ 1 | Number of replicas (optional, default 1) |
| `dataPlaneConfig` | object | — | Data plane configuration (optional) |

### Edge Model

Each edge represents a communication path:

| Property | Type | Range | Description |
|---|---|---|---|
| `id` | string | — | Unique identifier |
| `source` | string | — | Source node ID |
| `target` | string | — | Target node ID |
| `type` | EdgeType | 4 types | Communication pattern |
| `latencyDistribution` | number | ≥ 0 ms | Average communication latency |
| `packetLossProbability` | number | 0–100% | Probability of message loss |
| `retryPolicy` | number | ≥ 0 | Retry attempts on this edge |
| `timeoutThreshold` | number | ≥ 0 ms | Timeout before failure |

## Node Type Catalog

### Edge Layer
- **CDN** — Content delivery network edge node
- **Edge Cache** — Regional edge cache
- **Geo DNS / Traffic Router** — Geographic traffic routing

### Ingress Layer
- **API Gateway** — API routing, auth, rate limiting
- **Load Balancer** — L4/L7 traffic distribution
- **Reverse Proxy** — Request forwarding and TLS termination
- **WAF** — Web application firewall

### Compute Layer
- **Stateless Service** — Horizontally scalable compute
- **Stateful Service** — Services with local state (sessions, caches)
- **Background Worker** — Async task processing
- **Batch Processor** — Scheduled batch workloads

### Data Layer
- **Database (Primary)** — Primary data store
- **Replica Set** — Read replicas
- **Cache** — In-memory cache (Redis, Memcached)
- **Distributed KV Store** — Distributed key-value store
- **Search Index** — Full-text search engine

### Messaging Layer
- **Message Broker** — Message queue (Kafka, RabbitMQ)
- **Stream Processor** — Stream processing engine
- **Dead Letter Queue** — Failed message sink

### Mesh Layer
- **Service Mesh Proxy** — Sidecar proxy (Envoy, Istio)

### Egress Layer
- **External API** — Third-party API dependency
- **Payment Gateway** — Payment processing service
- **Third-Party Service** — External service dependency

## Edge Types

| Type | Description | Use Case |
|---|---|---|
| `sync` | Synchronous request/response | HTTP, gRPC calls |
| `async` | Asynchronous event delivery | Message queue publishing/consuming |
| `replication` | Data replication | Primary → replica data sync |
| `mesh-hop` | Service mesh routing | Sidecar-to-sidecar communication |

## Constraints

- **Max 40 nodes** per architecture
- **Max 80 edges** per architecture
- No self-loops (source ≠ target)
- Unique IDs for all nodes and edges
- File import limit: 2MB

## Graph Layout

The Architecture Graph View organizes nodes into horizontal zones by category:

```
Edge → Ingress → Compute → Messaging → Data → Mesh → Egress
```

Each zone uses vertical spacing to prevent overlap. Edges are rendered as curved paths with type-specific styling (solid for sync, dashed for async, dotted for replication, dash-dot for mesh-hop).

## Templates

OrchEngineX includes starter templates for common architectures:

| Template | Nodes | Description |
|---|---|---|
| Microservices Basic | 8 | Gateway, LB, 3 services, DB, replica, cache |
| Event-Driven Pipeline | 16 | Full async pipeline with message broker and workers |
| Multi-Region HA | 18 | Geo-DNS, multi-region with cross-region replication |
| Full-Stack HFT | 22 | Complete HFT stack across all 7 layers |

See the [`examples/`](../examples/) directory for importable JSON files.
