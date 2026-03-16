# CLI Reference

The `oex` CLI provides full command-line access to OrchEngineX. Distributed as the `@orchenginex/cli` npm package.

## Installation

```bash
npm install -g @orchenginex/cli
```

## Authentication

```bash
# Browser-based OAuth (opens browser, 120s timeout)
oex login

# Token-based (for CI/CD)
oex login --token <api-token>

# Check current session
oex whoami
```

Credentials are stored in `~/.oex/credentials.json` with `0o600` permissions.

Environment variable override: `OEX_API_TOKEN`.

## Commands

### `oex init`

Initialize a new project with a starter architecture template.

```bash
oex init
# Interactive prompts for template selection
```

Creates a `.oex.yml` project manifest and starter `architecture.json`.

### `oex validate <file>`

Offline validation of architecture JSON against the schema.

```bash
oex validate ./architecture.json
```

Checks:
- Schema compliance (node types, edge types, required fields)
- Constraint enforcement (max 40 nodes, max 80 edges)
- Structural integrity (no self-loops, valid references)
- Numeric range clamping

### `oex arch list`

List all saved architectures.

```bash
oex arch list
```

Output columns: ID, Name, Nodes, Edges, Updated.

### `oex arch import <file>`

Import architecture from a JSON file.

```bash
oex arch import ./my-arch.json --name "Production" --description "Current prod topology"
```

Options:
- `--name <n>` — Override architecture name
- `--description <desc>` — Set description

### `oex arch inspect <id>`

Show architecture summary with structural analysis.

```bash
oex arch inspect abc123-def456
```

Output includes:
- Node/edge counts, density, fan-out index
- Critical path depth, spectral radius
- **Fragility score** (identical formula to web UI)
- Cascade susceptibility, partition sensitivity, retry amp risk
- SPOF detection

### `oex arch delete <id>`

Delete a saved architecture.

```bash
oex arch delete abc123 --force
```

Options:
- `--force` — Skip confirmation prompt

### `oex simulate`

Run a failure simulation.

```bash
oex simulate \
  --arch <architecture-id> \
  --failure-node db-primary \
  --latency 150 \
  --packet-loss 5 \
  --retries 3
```

Required: `--failure-node`
Optional: `--arch` (uses architecture from API), `--latency`, `--packet-loss`, `--retries`

### `oex fragility score <archId>`

Get the current fragility score.

```bash
oex fragility score abc123
```

Output:
- Fragility score (color-coded)
- Topology, mechanics, data plane contributions
- Node/edge counts
- Snapshot timestamp

### `oex fragility trend <archId>`

View historical fragility snapshots.

```bash
oex fragility trend abc123 --limit 20
oex fragility trend abc123 --output trend.csv --format csv
oex fragility trend abc123 --output trend.json --format json
```

Options:
- `--limit <n>` — Max snapshots (default: 20)
- `--output <path>` — Export to file
- `--format <fmt>` — `json` (default) or `csv`

### `oex analyze structure <file|id>`

Run structural analysis on a local file or remote architecture.

```bash
oex analyze structure ./architecture.json
oex analyze structure abc123
```

### `oex experiments run`

Run a matrix-based parameter sweep.

```bash
oex experiments run --arch <architecture-id>
```

Options:
- `--arch <id>` — Architecture to sweep against

### `oex experiments list`

List all experiment runs.

```bash
oex experiments list
```

### `oex experiments report <id>`

Generate and save an experiment report.

```bash
oex experiments report exp-123
# Saves to experiment-exp-123-report.json
```

### `oex export <type> <id>`

Export data in various formats.

```bash
oex export architecture abc123 --format json
oex export architecture abc123 --format yaml
oex export simulation sim-456 --format csv
oex export fragility abc123 --format csv
```

Types: `architecture`, `simulation`, `fragility`
Formats: `json`, `yaml`, `csv`

### `oex visualize <type> <id>`

Generate visual artifacts.

```bash
oex visualize graph abc123 --format svg --output arch.svg
oex visualize graph abc123 --format png --output arch.png
oex visualize heatmap abc123 --format svg
oex visualize trend abc123 --format svg
```

Types: `graph`, `heatmap`, `trend`
Formats: `svg`, `png`, `html`

## Rate Limits

| Endpoint | Limit | Headers |
|---|---|---|
| `/simulate` | 30/min | `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` |
| `/visualize` | 20/min | Same |
| `/data-api` | 60/min | Same |
| `/experiments` | 10/min | Same |

429 responses include retry duration information.

## Error Handling

All commands handle `401 Unauthorized` by prompting: `Session expired. Run 'oex login' to re-authenticate.`

## Configuration Files

| File | Location | Description |
|---|---|---|
| `.oex.yml` | Project root | Project manifest |
| `credentials.json` | `~/.oex/` | Auth credentials (0o600) |

## Environment Variables

| Variable | Description |
|---|---|
| `OEX_API_TOKEN` | Authentication token (overrides credentials file) |
| `OEX_API_URL` | API base URL (defaults to production) |
