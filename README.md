# USDAX Finance — API Server

Backend REST API for the USDAX Finance protocol interface. Reads live data from deployed smart contracts on Robinhood Chain, stores vault and yield positions in PostgreSQL, and exposes a typed OpenAPI-compatible HTTP interface consumed by the frontend.

---

## Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js 20+ (ESM) |
| Framework | Express 5 |
| Database | PostgreSQL + Drizzle ORM |
| Chain reads | viem v2 (read-only, no wallet) |
| Logging | Pino |
| Build | esbuild |
| Schema validation | Zod |

---

## API Endpoints

### Protocol
| Method | Path | Description |
|---|---|---|
| GET | `/api/health` | Health check |
| GET | `/api/config` | Frontend config (contract addresses, chain ID) |
| GET | `/api/protocol/stats` | Total collateral, USDAX supply, active vaults, TVL |
| GET | `/api/protocol/activity` | Recent protocol activity feed |
| GET | `/api/protocol/collateral-breakdown` | TVL breakdown by collateral token |
| GET | `/api/protocol/health-distribution` | Distribution of vault health factors |

### Positions (Vaults)
| Method | Path | Description |
|---|---|---|
| GET | `/api/positions` | List all positions (filter: `?owner=`, `?status=`) |
| POST | `/api/positions` | Open a new vault position |
| GET | `/api/positions/:id` | Get a single position |
| PATCH | `/api/positions/:id` | Update position state |
| DELETE | `/api/positions/:id` | Close a position |

### Yield
| Method | Path | Description |
|---|---|---|
| GET | `/api/yield/pools` | List all yield pools |
| GET | `/api/yield/stats` | Aggregate yield stats (filter: `?owner=`) |
| GET | `/api/yield/positions` | List yield positions (filter: `?owner=`) |
| POST | `/api/yield/deposit` | Record a new yield deposit |
| POST | `/api/yield/withdraw/:id` | Record a yield withdrawal |
| POST | `/api/yield/claim/:id` | Record a reward claim |

### Liquidations
| Method | Path | Description |
|---|---|---|
| GET | `/api/liquidations` | List liquidation events |
| POST | `/api/liquidations` | Record a liquidation |

### Staking
| Method | Path | Description |
|---|---|---|
| GET | `/api/staking/stats` | Staking stats (projected — APX not yet deployed) |
| GET | `/api/staking/positions` | Staking positions (filter: `?owner=`) |

---

## Database Schema

### `positions`
Tracks CDP vault positions opened through VaultEngine.

| Column | Type | Description |
|---|---|---|
| `id` | serial | Primary key |
| `owner` | text | EIP-55 checksummed wallet address |
| `collateral_token` | text | WETH / WBTC / stETH |
| `collateral_amount` | numeric(30,18) | Raw token amount |
| `collateral_value_usd` | numeric(30,6) | USD value at open |
| `usdax_minted` | numeric(30,6) | USDAX debt (full amount, pre-fee) |
| `health_factor` | numeric(30,18) | HF at last update |
| `collateral_ratio` | numeric(10,4) | Collateral / debt ratio |
| `status` | text | `active` / `closed` / `liquidated` |

### `yield_positions`
Tracks USDAX deposits in USDAxSavings.

| Column | Type | Description |
|---|---|---|
| `id` | serial | Primary key |
| `owner` | text | Wallet address |
| `pool_id` | text | Pool identifier (`usdax-savings`) |
| `deposited_usdax` | numeric(30,18) | Principal deposited |
| `status` | text | `active` / `withdrawn` |
| `last_claim_at` | timestamp | Last reward claim checkpoint |
| `total_claimed_usdax` | numeric(30,18) | Cumulative rewards claimed |

---

## Getting Started

### Prerequisites
- Node.js 20+
- pnpm — `npm install -g pnpm`
- PostgreSQL 14+

### Install
```bash
pnpm install
```

### Configure environment
```bash
cp .env.example .env
```

Fill in `.env` with your values (see `.env.example` for all required vars).

### Push database schema
```bash
pnpm db:push
```

### Run dev server
```bash
pnpm dev
```

Server starts on `PORT` (default 3001).

### Build for production
```bash
pnpm build
node dist/index.mjs
```

---

## Environment Variables

See `.env.example` for a full list. Required variables:

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `PORT` | HTTP server port |
| `RPC_URL` | Robinhood Chain RPC endpoint |
| `CONTRACT_USDAX` | USDAxToken address |
| `CONTRACT_VAULT_ENGINE` | VaultEngine address |
| `CONTRACT_COLLATERAL_MANAGER` | CollateralManager address |
| `CONTRACT_ORACLE` | MockPriceOracle address |
| `CONTRACT_SAVINGS` | USDAxSavings address |
| `CONTRACT_WETH` | WETH token address |
| `CONTRACT_WBTC` | WBTC token address |
| `CONTRACT_STETH` | stETH token address |

---

## Related Repositories

- [USDAX-Finance/contracts](https://github.com/USDAX-Finance/contracts) — Foundry smart contracts
- [USDAX-Finance/interface](https://github.com/USDAX-Finance/interface) — React frontend

---

## License

MIT
