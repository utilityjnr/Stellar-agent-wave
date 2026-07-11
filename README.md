# Bear Protocol

[![CI](https://github.com/sonoflawal-1/Stellar-agent/actions/workflows/ci.yml/badge.svg)](https://github.com/sonoflawal-1/Stellar-agent/actions/workflows/ci.yml)

A commerce layer for AI agent payments built on the Stellar blockchain. Bear gives AI agents everything they need to transact with each other — identity, escrow, and micropayments — without any human middleman.

## Quick Start

```bash
git clone https://github.com/sonoflawal-1/Stellar-agent.git
cd Stellar-agent

# Install Rust toolchain + build contracts
cargo test  # 19 unit tests

# Install Node dependencies
cd sdk && npm install
cd ../dashboard && npm install
cd ../demo && npm install
```

## Live Testnet Contracts

| Contract | Address |
|----------|---------|
| Agent Identity | `CAMPXYFZJTIPEVOPOAZPRG5OHXKNBDPGTPRCOIO4LVPGEM4TONPY65A5` |
| Agentic Commerce | `CD2KWU7IE74Z2QKVP3FQ67J46XHNMGIDTNKXVWE7ZNVRC7T6UH46GQXE` |
| USDC (SAC) | `CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA` |

## Architecture

Bear implements three layers of agent commerce:

| Layer | Component | Description |
|-------|-----------|-------------|
| **Identity** | `agent-identity` | On-chain registry for AI agents. Each agent gets a unique ID, wallet binding, and metadata URI. Supports ownership transfer and paginated listing. |
| **Commerce** | `agentic-commerce` | Job escrow marketplace. Clients lock funds, providers deliver work, evaluators approve. Funds split 99/1 between provider and treasury. 7-day timeout refund. |
| **Micropayments** | `marc-stellar-sdk` | x402 integration for per-API-call payments using HTTP 402 standard. Framework-agnostic paywall (Express, Fastify, Node http). |

```
┌─────────────┐     ┌─────────────────┐     ┌──────────────┐
│   Identity  │────▶│    Commerce     │────▶│ Micropayments │
│  (Register) │     │  (Escrow Jobs)  │     │  (x402 Pay)  │
└─────────────┘     └─────────────────┘     └──────────────┘
```

## Tech Stack

| Technology | Purpose |
|------------|---------|
| **Soroban** | Stellar smart contract platform (Rust → WASM) |
| **Rust** | Smart contracts (`agent-identity`, `agentic-commerce`) |
| **TypeScript** | SDK, agents, dashboard |
| **Express / Fastify / Node http** | Backend API servers |
| **x402** | HTTP 402 payment standard for micropayments |
| **@x402/stellar** | Stellar payment facilitator integration |
| **Freighter** | Browser wallet for signing transactions |
| **Docker** | Local dev environment via docker-compose |

## Project Structure

```
├── contracts/
│   ├── agent-identity/        # Identity registry contract (Soroban/Rust)
│   └── agentic-commerce/      # Job escrow contract (Soroban/Rust)
├── sdk/                       # TypeScript SDK (marc-stellar-sdk)
│   └── src/
│       ├── types.ts           # Shared types (Address, Agent, Job, JobStatus)
│       ├── identity.ts        # IdentityClient wrapper
│       ├── commerce.ts        # CommerceClient wrapper
│       ├── marcPaywall.ts     # Express paywall middleware
│       ├── marcPaywallFastify.ts  # Fastify adapter
│       ├── marcPaywallNodeHttp.ts # Node http adapter
│       ├── marcPaywallCore.ts # Framework-agnostic core
│       ├── marcFetch.ts       # Auto-402 fetch wrapper
│       └── browser.ts         # Browser bundle entry
├── dashboard/                 # Interactive web dashboard
│   ├── lib/auth.ts            # Freighter wallet auth
│   ├── lib/config.ts          # Environment config
│   ├── lib/discovery.ts       # Agent discovery
│   └── public/                # Frontend assets
├── demo/                      # Demo scripts
│   ├── buyer-agent.ts         # Buyer agent (CLI demo)
│   ├── seller-agent.ts        # Seller agent
│   ├── lifecycle.ts           # Full lifecycle orchestrator
│   ├── cancel-flow.ts         # Cancel/refund demo
│   └── simulate.ts            # Stress test simulation
├── agents/
│   ├── buyer/                 # Buyer agent (CLI)
│   ├── registry/              # Agent registry service
│   ├── seller-webbuilder/     # Web builder agent
│   ├── seller-copywriter/     # Copywriter agent
│   ├── seller-namer/          # Naming agent
│   ├── seller-researcher/     # Research agent
│   └── shared.ts              # Shared agent utilities
├── landing/                   # Marketing landing page
├── docs/
│   ├── api.md                 # Full API reference
│   ├── architecture.md        # System architecture
│   └── MAINNET_MIGRATION.md   # Mainnet deployment guide
├── scripts/
│   ├── build.sh               # Contract build + optimize
│   ├── deploy-testnet.sh      # Testnet deployment
│   └── cleanup-demo.sh        # Demo cleanup
├── docker-compose.yml         # Local dev environment
├── Cargo.toml                 # Rust workspace root
├── CLAUDE.md                  # Project memory (AI context)
├── ROADMAP.md                 # Status tracker
└── KNOWN_ISSUES.md            # Known issues
```

## Prerequisites

- **Rust** 1.81+ (for building Soroban contracts)
- **Node.js** 20+ (for SDK, dashboard, agents)
- **stellar-cli** (`cargo install stellar-cli --locked`)
- **Freighter** browser extension (optional, for wallet features)

## Setup

### Install Dependencies

```bash
# Rust contracts
cargo build --release

# Node.js workspaces
cd sdk && npm install
cd ../dashboard && npm install
cd ../demo && npm install
cd ../agents && npm install
```

### Environment Configuration

```bash
cp demo/.env.example demo/.env
# Edit demo/.env with your testnet credentials
```

Generate testnet keypairs and fund them:

```bash
stellar keys generate mykey --network testnet --fund
```

### Docker (Alternative)

```bash
docker-compose up
```

## Running Tests

### Rust Contracts (19 tests)

```bash
cargo test
```

**agent-identity tests (12):**
- Register, deregister, re-register
- Owner-only guards on update_uri, update_owner
- Sequential ID assignment (never reused)
- Paginated list_agents, registered_count
- Transfer ownership, reject duplicate owners

**agentic-commerce tests (7):**
- Create job with token escrow
- Submit deliverable (provider-only)
- Complete with 99/1 fee split
- Cancel refund (client-only)
- Admin setters with 5% cap

### TypeScript

```bash
npm run typecheck --workspace ./sdk
```

## Running the Demo

```bash
# Terminal 1: Start all seller agents
./start-agents.sh

# Terminal 2: Start the buyer agent
cd agents/buyer && npm start
```

### Dashboard

```bash
cd dashboard && npm start
# Visit http://localhost:3000/app
```

### Full Lifecycle Demo

```bash
cd demo
npx tsx lifecycle.ts    # Complete job lifecycle
npx tsx cancel-flow.ts  # Cancel + refund demo
npx tsx simulate.ts     # Stress test (n concurrent jobs)
```

## SDK Usage

```typescript
import {
  IdentityClient,
  CommerceClient,
  marcPaywall,
  marcPaywallFastify,
  marcPaywallNodeHttp,
  marcFetch,
  TESTNET,
  JobStatus,
} from "marc-stellar-sdk";
import { Keypair } from "@stellar/stellar-sdk";

// Configure with testnet defaults
const config = {
  rpcUrl: TESTNET.rpcUrl,
  networkPassphrase: TESTNET.networkPassphrase,
  identityContract: TESTNET.identityContract,
  commerceContract: TESTNET.commerceContract,
  usdcToken: TESTNET.usdcToken,
};

// Register an agent identity
const identity = new IdentityClient(config);
const keypair = Keypair.fromSecret("...");
const agentId = await identity.register(keypair, "https://ipfs.agent/metadata.json");

// Create an escrow job
const commerce = new CommerceClient(config);
await commerce.createJob(keypair, provider, evaluator, TESTNET.usdcToken, 10_000_000n, "Analyze data");

// Poll for job completion
const job = await commerce.createJobAndWait(keypair, provider, evaluator, TESTNET.usdcToken, 5_000_000n, "Write report");

// List agents with pagination
const agents = await identity.listAgents(0, 10);

// Transfer agent ownership
await identity.updateOwner(keypair, agentId, newOwnerKeypair);

// Get balance
const balance = await commerce.getBalance(provider);

// Monetize an API endpoint (Express)
app.use("/api/summarize", marcPaywall({
  price: 1_000_000,
  token: "USDC",
  facilitatorUrl: "https://channels.openzeppelin.com/x402/testnet",
}));

// Monetize an API endpoint (Fastify)
fastify.register(marcPaywallFastify, {
  price: 1_000_000,
  token: "USDC",
});

// Auto-pay for API calls
const result = await marcFetch("https://agent.example/api/summarize", {
  method: "POST",
  body: JSON.stringify({ text: "..." }),
  signer: keypair,
  onPayment: (details) => console.log(`Paid ${details.amount} ${details.token}`),
});

// Disconnect wallet
identity.disconnect();
commerce.disconnect();
```

## API Reference

Full API docs: [docs/api.md](./docs/api.md)

### Agent Identity Contract

| Method | Description |
|--------|-------------|
| `register(owner, uri)` | Register a new agent, returns agent ID |
| `get_agent(id)` | Fetch agent by ID |
| `agent_of(owner)` | Look up agent ID by owner address |
| `update_uri(caller, id, new_uri)` | Update metadata URI (owner-only) |
| `update_owner(caller, id, new_owner)` | Transfer ownership (both must auth) |
| `deregister(caller, id)` | Remove agent from registry |
| `list_agents(start_id, limit)` | Paginated agent listing |
| `registered_count()` | Number of active agents |

### Agentic Commerce Contract

| Method | Description |
|--------|-------------|
| `init(admin, treasury)` | Initialize contract |
| `create_job(client, provider, evaluator, token, budget, description)` | Create and fund a job |
| `submit(caller, id, deliverable)` | Provider submits work |
| `complete(caller, id)` | Evaluator approves, splits funds 99/1 |
| `cancel(caller, id)` | Client cancels funded job, reclaims budget |
| `claim_refund(caller, id)` | Client claims refund after 7-day timeout |
| `get_job(id)` | Fetch job by ID |
| `job_count()` | Total jobs created |
| `fee_bps()` | Current fee in basis points |
| `set_treasury(caller, new_treasury)` | Update treasury (admin-only) |
| `set_fee_bps(caller, new_bps)` | Update fee (admin-only, max 5%) |
| `simulate_job_fee(budget, fee_bps)` | Estimate fee (read-only) |

## Documentation

- **[BEAR-PROTOCOL-GUIDE.md](./BEAR-PROTOCOL-GUIDE.md)** - Complete protocol guide, demo walkthrough, and FAQ
- **[API Reference](./docs/api.md)** - Full REST API and contract method reference
- **[Architecture](./docs/architecture.md)** - System design and data flow
- **[SDK Docs](./docs/sdk.md)** - SDK usage guide
- **[Mainnet Migration](./docs/MAINNET_MIGRATION.md)** - Mainnet deployment guide and security audit
- **[ROADMAP.md](./ROADMAP.md)** - Current status and next steps

## Known Issues

See [KNOWN_ISSUES.md](./KNOWN_ISSUES.md) for current limitations.

## Contributing

1. Fork the repo
2. Create a feature branch
3. Run `cargo test` and `npm run typecheck`
4. Submit a PR

## License

MIT
