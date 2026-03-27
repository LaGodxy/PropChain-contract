# PropChain Development Environment Setup

> **Documentation Version**: 2.0.0 — Updated March 2026
> Covers all contracts shipped in release v2.x. See [Documentation Maintenance](docs/documentation-maintenance.md) for the versioning policy.

This guide will help you set up a complete development environment for PropChain smart contracts.

## Quick Start

```bash
# Clone and setup
git clone https://github.com/MettaChain/PropChain-contract.git
cd PropChain-contract
./scripts/setup.sh

# Start local development environment
docker-compose up -d

# Run tests
./scripts/test.sh

# Build contracts
./scripts/build.sh --release
```

## Prerequisites

- **Rust** 1.75+ with stable toolchain (see `rust-toolchain.toml`)
- **cargo-contract** 3.x for ink! smart contract development
- **Docker** and Docker Compose
- **Node.js** 18+ (for frontend / SDK development)
- **Git**

## Manual Setup

### 1. Install Rust and Tools

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env

# The repo's rust-toolchain.toml pins the exact stable channel automatically;
# just run any cargo command and rustup will download the right toolchain.

# Install cargo-contract (ink! CLI)
cargo install cargo-contract --locked --version "^3"

# Add WASM compile target
rustup target add wasm32-unknown-unknown

# Optional but recommended
cargo install cargo-deny cargo-audit
```

### 2. Setup Pre-commit Hooks

```bash
./scripts/setup-pre-commit.sh
```

### 3. Start Local Development

```bash
# Start blockchain node
./scripts/local-node.sh start

# Or use Docker Compose for full stack
docker-compose up -d
```

## Development Workflow

### Building Contracts

```bash
# Debug build
./scripts/build.sh

# Release build
./scripts/build.sh --release

# Clean build
./scripts/build.sh --clean
```

### Running Tests

```bash
# All tests
./scripts/test.sh

# Unit tests only
./scripts/test.sh --no-integration

# With coverage
./scripts/test.sh --coverage

# E2E tests
./scripts/e2e-test.sh
```

### Code Quality

```bash
# Format code
cargo fmt

# Run linting
cargo clippy

# Pre-commit checks
pre-commit run --all-files
```

### Deployment

```bash
# Local deployment
./scripts/deploy.sh --network local

# Testnet deployment
./scripts/deploy.sh --network westend

# Mainnet deployment
./scripts/deploy.sh --network polkadot
```

## Project Structure

```
PropChain-contract/
├── contracts/                  # Smart contract source code
│   ├── ai-valuation/           # On-chain AI property valuation oracle
│   ├── analytics/              # Event analytics aggregator
│   ├── bridge/                 # Cross-chain asset bridge
│   ├── compliance_registry/    # KYC/AML compliance registry
│   ├── escrow/                 # Escrow and settlement engine
│   ├── fees/                   # Dynamic fee calculation
│   ├── fractional/             # Fractional ownership shares
│   ├── governance/             # DAO governance and voting
│   ├── insurance/              # Property insurance pools
│   ├── ipfs-metadata/          # IPFS metadata pointer contract
│   ├── lib/                    # Shared library code & trait implementations
│   ├── oracle/                 # Price and property data oracle
│   ├── prediction-market/      # Property price prediction market
│   ├── property-management/    # Property lifecycle management
│   ├── property-token/         # ERC-721 property NFT (PSP34)
│   ├── proxy/                  # Upgradeable proxy pattern
│   ├── staking/                # PROP token staking
│   ├── traits/                 # Shared ink! trait interfaces
│   └── zk-compliance/          # Zero-knowledge compliance proofs
├── scripts/                    # Development and deployment scripts
├── tests/                      # Integration and E2E tests
├── docs/                       # Documentation (see docs/documentation-maintenance.md)
│   ├── adr/                    # Architecture Decision Records
│   ├── tutorials/              # Step-by-step guides
│   ├── architecture.md
│   ├── code-style-guide.md     # Code style reference
│   ├── contracts.md
│   ├── deployment.md
│   ├── incident-response.md    # Security incident procedures
│   ├── integration.md
│   ├── security_pipeline.md
│   ├── testing-guide.md
│   └── threat-model.md         # Threat model & mitigations
├── security-audit/             # Custom security scanner binary
├── .pre-commit-config.yaml     # Pre-commit hook configuration
├── clippy.toml                 # Clippy lint configuration
├── docker-compose.yml          # Local development stack
├── rust-toolchain.toml         # Rust version pin
└── rustfmt.toml                # Formatting configuration
```

## Environment Configuration

### Local Development (.env.local)

```env
NETWORK=local
NODE_URL=ws://localhost:9944
SURI=//Alice
```

### Testnet (.env.westend)

```env
NETWORK=westend
NODE_URL=wss://westend-rpc.polkadot.io
SURI=your-testnet-mnemonic
```

### Mainnet (.env.polkadot)

```env
NETWORK=polkadot
NODE_URL=wss://rpc.polkadot.io
SURI=your-mainnet-mnemonic
```

## Common Issues and Solutions

### Rust Installation Issues

```bash
# If Rust is not found
source ~/.cargo/env

# Update Rust toolchain
rustup update stable
```

### Contract Build Failures

```bash
# Clean build artifacts
cargo clean
rm -rf target/

# Rebuild
./scripts/build.sh --clean
```

### Node Connection Issues

```bash
# Check if node is running
curl http://localhost:9933/health

# Restart local node
./scripts/local-node.sh restart
```

### Pre-commit Hook Issues

```bash
# Reinstall hooks
./scripts/setup-pre-commit.sh --test-only

# Run hooks manually
pre-commit run --all-files
```

## IDE Configuration

### VS Code

Install these extensions:
- Rust Analyzer
- TOML Language Support
- Docker
- GitLens

### Workspace Settings (.vscode/settings.json)

```json
{
    "rust-analyzer.checkOnSave.command": "clippy",
    "rust-analyzer.cargo.loadOutDirsFromCheck": true,
    "editor.formatOnSave": true,
    "files.trimTrailingWhitespace": true
}
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests and linting
5. Submit a pull request

## Getting Help

- **Documentation**: Check the `docs/` directory
- **Issues**: [GitHub Issues](https://github.com/MettaChain/PropChain-contract/issues)
- **Discord**: [PropChain Community](https://discord.gg/propchain)
- **Email**: dev@propchain.io

## Next Steps

1. Read the [Architecture Guide](docs/architecture.md)
2. Follow the [Basic Property Registration Tutorial](docs/tutorials/basic-property-registration.md)
3. Explore the [Contract API](docs/contracts.md)
4. Set up your [Frontend Integration](docs/integration.md)
5. Read the [Code Style Guide](docs/code-style-guide.md)
6. Review the [Security Pipeline](docs/security_pipeline.md)

## Documentation Maintenance

Documentation is versioned alongside the code. When you add or change behaviour:

1. Update the relevant `docs/` file in the same PR.
2. Bump the `> **Documentation Version**` header in any file you change.
3. Add an entry to `docs/adr/` if you are making an architectural decision.

See [docs/documentation-maintenance.md](docs/documentation-maintenance.md) for the full process.
