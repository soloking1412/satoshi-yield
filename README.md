```markdown
# SatoshiYield

> Non-custodial sBTC yield optimizer on Stacks — see where your Bitcoin earns the most, rebalance in one click.

## Overview

SatoshiYield aggregates yield opportunities for sBTC holders across the Stacks DeFi ecosystem. Instead of manually checking Bitflow, ALEX, Zest, Velar, and Hermetica separately, SatoshiYield pulls live APY data from all of them, normalizes it, and lets you move your sBTC into the best position in a single transaction.

No custody. No token. Just a vault contract that works.

**Live protocols tracked:**
- Bitflow (22–50% APY on sBTC/STX pools)
- ALEX Lab (sBTC pools + Surge campaign rewards)
- Zest Protocol (1–2% extra BTC yield on sBTC supply)
- Velar (20%+ APY on STX-sBTC pair)
- Native sBTC holding (5% base APY from PoX stacking rewards)

---

## The Problem

sBTC holders on Stacks have 6+ places to earn yield right now. Rates change every cycle. Risk profiles are completely different across protocols. Most users just leave sBTC sitting at the 5% base rate because comparing everything manually is too much friction.

That's idle capital. SatoshiYield fixes the UX gap between "I have sBTC" and "my sBTC is working at maximum efficiency."

---

## Architecture

```
satoshi-yield/
├── contracts/
│   ├── vault.clar              # Core deposit/withdraw vault
│   ├── yield-tracker.clar      # Position and accrued yield tracking
│   ├── rebalancer.clar         # One-click rebalance execution logic
│   └── traits/
│       └── yield-source.clar   # Trait interface for protocol adapters
├── indexer/
│   ├── src/
│   │   ├── fetchers/
│   │   │   ├── bitflow.ts      # Bitflow APY fetcher
│   │   │   ├── alex.ts         # ALEX APY fetcher
│   │   │   ├── zest.ts         # Zest APY fetcher
│   │   │   └── velar.ts        # Velar APY fetcher
│   │   ├── normalizer.ts       # Normalize APY data to common format
│   │   └── server.ts           # Express API server
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── YieldTable.tsx      # Live APY comparison table
│   │   │   ├── PositionCard.tsx    # User's active position display
│   │   │   ├── RebalanceModal.tsx  # One-click rebalance flow
│   │   │   └── WalletConnect.tsx   # Leather + Xverse wallet connection
│   │   ├── hooks/
│   │   │   ├── useVault.ts         # Vault contract read/write hooks
│   │   │   └── useYieldData.ts     # Indexer data hooks
│   │   └── App.tsx
│   └── package.json
├── tests/
│   ├── vault_test.ts           # Clarinet unit tests — vault contract
│   ├── rebalancer_test.ts      # Clarinet unit tests — rebalancer
│   └── integration/
│       └── full_flow_test.ts   # Deposit → track → rebalance flow
└── docs/
    ├── SECURITY.md             # Threat model and known assumptions
    └── AUDIT_NOTES.md          # Internal audit findings and resolutions
```

---

## Smart Contracts

### vault.clar
Core non-custodial vault. Handles sBTC deposits and withdrawals. Tracks each user's principal separately. Takes a **5% performance fee on yield only** — never touches principal.

```clarity
;; Core functions
(define-public (deposit (amount uint)))
(define-public (withdraw (amount uint)))
(define-read-only (get-position (user principal)))
(define-read-only (get-accrued-yield (user principal)))
```

### yield-tracker.clar
Reads current APY from each integrated protocol adapter. Computes the best available rate at any given block. Used by the frontend to power the comparison table.

### rebalancer.clar
Executes the actual position move. Withdraws from current protocol, deposits into target protocol, updates position record — all in one atomic Clarity call.

### yield-source trait
```clarity
(define-trait yield-source-trait
  (
    (get-current-apy () (response uint uint))
    (deposit-sbtc (uint) (response bool uint))
    (withdraw-sbtc (uint) (response bool uint))
  )
)
```
Each protocol integration implements this trait. Adding a new protocol = writing one new adapter contract. The core vault never changes.

---

## Fee Model

| Fee type | Amount | When charged |
|---|---|---|
| Performance fee | 5% of yield only | On claim / rebalance |
| Deposit fee | None | — |
| Withdrawal fee | None | — |
| Principal fee | None | Never |

At $500K TVL earning 15% average APY → ~$3,750/year in protocol revenue. Self-sustaining without further grants at that scale.

---

## Milestones

### Milestone 1 — Testnet (Week 4)
- [ ] Vault, yield-tracker, rebalancer contracts deployed on Stacks testnet
- [ ] Indexer live and returning APY data from Bitflow + ALEX
- [ ] Clarinet unit tests passing for core vault logic
- [ ] Testnet contract addresses published here

**Evidence:** Testnet contract addresses on Stacks Explorer + passing Clarinet test output in `/tests`

### Milestone 2 — Beta (Week 8)
- [ ] Frontend live on testnet at beta.satoshiyield.xyz
- [ ] Leather + Xverse wallet connection working
- [ ] Deposit, position view, and rebalance flow functional end-to-end
- [ ] Zest and Velar adapters added to indexer
- [ ] Community beta open — first 20 testers onboarded via Stacks Discord

**Evidence:** Live beta URL + on-chain testnet transactions from real testers

### Milestone 3 — Mainnet Launch (Week 12)
- [ ] All contracts audited (internal) and deployed to mainnet
- [ ] satoshiyield.xyz live with full UI
- [ ] DefiLlama listing submitted for TVL tracking
- [ ] $50K+ TVL within 2 weeks of launch
- [ ] Full audit notes published in `/docs/AUDIT_NOTES.md`

**Evidence:** Mainnet contract addresses + DefiLlama TVL data + real deposit transactions on Stacks Explorer

---

## Tech Stack

| Layer | Tech |
|---|---|
| Smart contracts | Clarity 4 |
| Contract testing | Clarinet |
| Indexer | Node.js + TypeScript |
| Frontend | React + Stacks.js |
| Wallet support | Leather, Xverse via WalletConnect |
| Hosting | Vercel (frontend) + Railway (indexer) |

---

## Security Approach

I'm a professional smart contract auditor (15+ audits on Sherlock, Code4rena, Immunefi). SatoshiYield's contracts go through the same process I'd apply to a paid client engagement:

- Threat modeling before writing a single line
- Invariant testing: principal never decreases, fee never exceeds 5% of yield, withdrawals always match deposits minus fees
- Clarity 4 is decidable by design — no reentrancy, no unbounded loops
- All contracts open source from day one
- `/docs/SECURITY.md` documents known assumptions and trust boundaries
- Community testnet beta with capped deposits before mainnet open

---

## Progress Links

| Item | Link |
|---|---|
| GitHub repo | github.com/[soloking1412]/satoshi-yield |
| Testnet contracts | *(updated at Milestone 1)* |
| Beta frontend | *(updated at Milestone 2)* |
| Mainnet contracts | *(updated at Milestone 3)* |
| DefiLlama TVL | *(updated at launch)* |

---

## Background

Built by a solo dev — Polkadot Blockchain Academy graduate, smart contract security auditor, and recipient of a $16,000 Arbitrum grant for the Stylus Developer Toolkit (CLI-first development environment for Arbitrum Stylus contracts). That project proved I can scope, build, and ship a grant-funded protocol alone on a fixed timeline. SatoshiYield is the same discipline applied to Stacks.

---

## Status

`[IN DEVELOPMENT]` — Stacks Endowment Getting Started Grant application in progress. Contracts in active development on local Clarinet environment. Testnet deploy targeted for Week 4.

---

## License

MIT — fully open source.
