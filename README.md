# Plasma Mainnet Non-Validator Template

Docker Compose template for running a Plasma mainnet non-validator (observer) node.

## Prerequisites

- Docker and Docker Compose installed
- SSD storage with ~400 GB free for mainnet sync
- Sufficient RAM to avoid swap usage

## Quick Start

```bash
# Clone
git clone https://github.com/PlasmaLaboratories/non-validator-templates.git
cd non-validator-templates

# Start the node
docker compose up -d

# Verify
docker compose ps
docker compose logs -f plasma-consensus
```

## Directory Structure

```
├── docker-compose.yml            # Service definitions
├── .env                          # Image versions and configuration
├── non-validator.toml            # Consensus configuration
├── enodes.txt                    # Execution bootstrap nodes
├── shared/                       # Validator keys & identities (read-only)
│   ├── mainnet.json              # Chain spec
│   ├── keys/                     # BLS12-381 validator public keys
│   └── identities/               # Validator identity files
└── data/                         # Local data directory (created on first run)
    ├── consensus/
    ├── execution/
    ├── jwt/
    └── genesis/
```

## Configuration

All version numbers and image tags are defined in `.env` — the single source of truth.

### Environment Variables (`.env`)

| Variable | Default | Description |
|----------|---------|-------------|
| `DATA_DIR` | `./data` | Local directory for consensus, execution, JWT, and genesis data |
| `NETWORK` | `mainnet` | Network chain identifier |
| `PLASMA_CONSENSUS_IMAGE` | `ghcr.io/plasmalaboratories/plasma-consensus-public` | Consensus image |
| `PLASMA_CONSENSUS_TAG` | `0.15.0` | Consensus image tag |
| `PLASMA_EXECUTION_IMAGE` | `ghcr.io/paradigmxyz/reth` | Execution image |
| `PLASMA_EXECUTION_TAG` | `v1.8.3` | Execution image tag |

### Consensus Configuration (`non-validator.toml`)

| Section | Fields | Description |
|---------|--------|-------------|
| *(top-level)* | `engine_api_url`, `consensus_api_host`, `authrpc_jwtsecret` | Execution engine connection |
| `[persistence]` | `data_dir` | Consensus data storage path |
| `[network]` | `p2p_port`, `interval`, `timeout`, `identity_file_path` | P2P networking |
| `[api]` | `enabled`, `host`, `port` | Consensus API endpoint |
| `[validators.*]` | `validator_keystore_pk_file_path`, `identity_file_path` | Validator committee |
| `[network.bootstrap_nodes.*]` | `api_host`, `p2p_port`, `peer_id` | Consensus bootstrap peers |

### External Address (NAT)

For nodes behind NAT, add to `non-validator.toml`:

```toml
[network]
external_address = "node.example.com:34070"
```

Or via CLI: `--p2p.external-address node.example.com:34070`

### Ports

| Service | Port | Protocol | Description |
|---------|------|----------|-------------|
| HTTP RPC | 8545 | HTTP | JSON-RPC API |
| WebSocket | 8546 | WS | WebSocket RPC |
| Execution metrics | 9001 | HTTP | Prometheus metrics |

## Usage

```bash
docker compose up -d                              # Start
docker compose logs -f                            # Logs
docker compose down                               # Stop
docker compose down && docker compose up -d       # Restart
```

## Troubleshooting

### Container Startup Failures
```bash
docker compose logs <service-name>
```

### Image Pull Issues
```bash
docker pull ghcr.io/plasmalaboratories/plasma-consensus-public:0.15.0
```

### Sync Status
```bash
curl -s -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' \
  http://localhost:8545
```

## Performance

- Use SSD storage for optimal I/O
- Ensure sufficient RAM to avoid swap usage
- Monitor CPU usage during initial sync
- Consider increasing ulimits for production deployments
