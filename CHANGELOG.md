# Changelog

All notable changes to OrchEngineX are documented here.

## [0.9.0] — 2026-03-14

### Added
- **Fragility Trend Persistence** — snapshots now persist to database and survive page reloads
- **Mechanics Analysis Panel** — styled node health dashboard with aggregate stats, color-coded health bars, and degradation tooltips
- **Fragility Score Alignment** — CLI (`oex arch inspect`, `oex fragility score`) and web UI now use identical composite formula
- **Technical Documentation Hub** — 14-section reference accessible at `/docs` and via Lab Divisions

### Fixed
- Fragility score mismatch between CLI API and Architecture Builder (unified 6-factor weighted composite)
- Unstyled mechanics analysis output now renders as a proper dashboard

## [0.8.0] — 2026-03-12

### Added
- **CLI** — `oex` command-line interface with 11 core commands
  - `login`, `whoami`, `init`, `validate`, `simulate`, `arch`, `fragility`, `analyze structure`, `experiments`, `export`, `visualize`
- **CLI Authentication** — browser-based OAuth with 120s timeout and `--token` fallback
- **Experiments API** — matrix-based parameter sweep simulations via `/experiments` endpoint
- **Visualization API** — SVG, PNG, and interactive HTML generation via `/visualize` endpoint
- **Rate Limiting** — IP-based rate limiting with `X-RateLimit-*` headers on all public endpoints

## [0.7.0] — 2026-03-09

### Added
- **Queue Collapse Case Study** — "The 4-Minute Queue Collapse" publication with interactive simulation
- **Queue Collapse Simulation** — configurable traffic multiplier, consumer count, processing time, retry limits
- **Queue Architecture Graph** — 19-node event-driven topology visualization

## [0.6.0] — 2026-03-06

### Added
- **Pool Saturation Cascade Case Study** — "How a 2% Latency Spike Collapses a 20-Service System" with 3 interactive simulations
- **Pool Saturation Simulation** — connection pool exhaustion modeling with baseline, spike, and mitigation modes
- **Publications Hub** — research index with category filtering

## [0.5.0] — 2026-03-01

### Added
- **Growth Mode Analysis** — scaling efficiency prediction with coordination overhead modeling
- **Data Plane Simulator** — consistency/replication/sharding/transaction impact modeling
- **Comparison View** — side-by-side simulation result comparison
- **Architecture Export** — SVG, PNG, and MP4 export from graph views

## [0.4.0] — 2026-02-20

### Added
- **System Mechanics Simulator** — 6 composable engines (compute, consistency, replication, transaction, distribution, flow control)
- **Mechanics ↔ Architecture bidirectional sync** — mechanics configs derived from architecture nodes and applied back
- **Node Health Scoring** — per-node health assessment based on failure probability, resource capacity, latency, and type

## [0.3.0] — 2026-02-10

### Added
- **Architecture Persistence** — save/load architectures to database with user authentication
- **Fragility Tracking** — historical fragility score snapshots with trend visualization
- **JSON Import** — validated architecture import with schema enforcement and lenient partial imports
- **Architecture Templates** — starter templates (microservices, event-driven, multi-region, full-stack)

## [0.2.0] — 2026-02-01

### Added
- **Structural Analysis** — fan-out index, critical path depth, dependency density, spectral radius approximation
- **Risk Indicators** — cascade susceptibility, partition sensitivity, quorum fragility, retry storm potential, SPOF detection
- **Node Removal Simulation** — availability drop, latency shift, retry amplification delta, partition risk delta
- **Architecture Graph Visualization** — zone-based layout with node status indicators and edge type legend

## [0.1.0] — 2026-01-15

### Added
- **Architecture Builder** — visual DWFG editor with 22 node types and 4 edge types
- **Initial node type catalog** — edge, ingress, compute, data, messaging, mesh, egress layers
- **User Authentication** — email/password signup and login
- **Landing Page** — platform overview with lab divisions
