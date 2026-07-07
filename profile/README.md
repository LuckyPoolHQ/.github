<div align="center">

# 🍀 LuckyPool

### No-loss prize savings on Stellar.

**Deposit USDC → Earn real yield via Blend → One winner takes the yield every week → Your principal never leaves your side.**

[Contracts](https://github.com/LuckyPoolHQ/LuckyPool-contracts) · [Frontend](https://github.com/LuckyPoolHQ/LuckyPool-frontend) · [SDK](https://github.com/LuckyPoolHQ/LuckyPool-sdk) · [Docs](https://github.com/LuckyPoolHQ/LuckyPool-docs)

</div>

---

## What is LuckyPool?

LuckyPool is a **no-loss prize savings protocol** on Stellar. Every depositor's principal is pooled and put to work earning yield through [Blend](https://blend.capital). Every week, all of that accumulated yield — and only the yield — is awarded to a single winner, selected by verifiable on-chain randomness. Nobody who doesn't win loses anything: deposits are always fully withdrawable.

Think of it as a lottery with the risk removed. This isn't a new idea — prize-linked savings schemes like the UK's **Premium Bonds** run on the same principle. Nothing like it exists in DeFi on Stellar today.

|  | |
|---|---|
| 💰 **Deposit** | Any amount of USDC. 1 USDC = 1 ticket toward the weekly draw. |
| 📈 **Earn** | Deposits are routed into Blend lending pools, earning yield continuously. |
| 🎟️ **Win or don't lose** | Each week's accumulated yield goes to one winner via VRF. Everyone else keeps 100% of their principal and rolls into the next round. |
| 🔓 **Withdraw anytime** | Principal is never locked or at risk. |

---

## Why this works on Stellar

Prize-linked savings needs two things most chains can't offer cheaply: frequent, low-friction transactions and access to real lending yield.

- Traditional savings accounts pay little in real terms
- Traditional lotteries extract a large share of every dollar wagered — arguably the worst financial product sold to ordinary people
- DeFi yield is real, but usually gas-expensive, complex, and intimidating to a non-native user

Stellar's sub-cent fees make weekly, permissionless draws and harvests economically viable at any deposit size — which is what makes the no-loss model actually work here.

---

## How it works

```text
 Depositor sends USDC
         │
         ▼
 LuckyPool contract records position, mints tickets (1 USDC = 1 ticket)
         │
         ▼
 USDC routed into a Blend lending pool → earns yield continuously
         │
         ▼
 harvest_yield()  (permissionless, weekly)
 Accrued yield moves from Blend → Prize Pool
         │
         ▼
 execute_draw()   (VRF-selected winner, weighted by ticket count)
 Winner receives the prize pool, minus a 5% protocol fee
 Everyone else: principal untouched, tickets carry into the next round
```

---

## Fair, verifiable draws

**The problem:** smart contracts are deterministic — every node must reach the same result, so a contract can't generate true randomness on its own. Naive approaches (block hash, ledger sequence, timestamp) are manipulable: a validator who is also a large depositor can see upcoming block data before it's finalized and choose to withhold a block that would make them lose. This is the classic MEV attack on on-chain lotteries.

**The solution:** LuckyPool uses a **Verifiable Random Function (VRF)**. A VRF takes a private key and a public input (the round number) and produces a random output plus a cryptographic proof. The contract verifies the proof on-chain before using the output — the VRF operator cannot manipulate the result without invalidating the proof. That gives the draw three properties a naive approach can't: unpredictable before it closes, verifiable after the fact, and resistant to manipulation by depositors, validators, or the team.

**Draw infrastructure:**

```text
 Friday midnight UTC
         │
         ▼
 harvest_yield()        (permissionless) — accrued yield moves from Blend into the prize pool
         │
         ▼
 request_draw()         (permissionless) — contract emits a VRF request to the oracle
         │
         ▼
 Oracle responds ~1–2 ledgers later with vrf_output + vrf_proof, posted on-chain
         │
         ▼
 execute_draw(vrf_output, vrf_proof)
   ├─ verifies the proof on-chain against the oracle's public key
   ├─ derives winning_ticket = vrf_output % total_tickets
   ├─ walks the depositor list to find who holds that ticket
   └─ pays out the prize pool, emits DrawCompleted { round, winner, prize, vrf_output }
```

Every draw emits enough data for anyone to independently re-verify the outcome after the fact — the VRF proof against the oracle's public key, the ticket arithmetic, and the winning address — using only public Stellar ledger history. Full design (VRF provider options, the commit-reveal fallback, the winner-selection algorithm) is in [`LuckyPool-docs/randomness.md`](https://github.com/LuckyPoolHQ/LuckyPool-docs/blob/main/randomness.md).

---

## Repositories

| Repository | Description |
|---|---|
| **[LuckyPool-contracts](https://github.com/LuckyPoolHQ/LuckyPool-contracts)** | Soroban smart contracts (Rust): deposits, Blend routing, yield harvesting, VRF-driven prize draw |
| **[LuckyPool-frontend](https://github.com/LuckyPoolHQ/LuckyPool-frontend)** | Next.js app — landing page and user dashboard (deposit, withdraw, lottery history) |
| **[LuckyPool-sdk](https://github.com/LuckyPoolHQ/LuckyPool-sdk)** | DrawEngine SDK — typed Soroban RPC client for reading/writing contract state |
| **[LuckyPool-docs](https://github.com/LuckyPoolHQ/LuckyPool-docs)** | Architecture, contract interface, yield integration, randomness design, go-to-market, pitch deck |

---

## Tech stack

| Layer | Technology |
|---|---|
| Smart contracts | Rust + Soroban SDK 26, deployed on Stellar |
| Yield source | [Blend Protocol](https://blend.capital) lending pools |
| Randomness | VRF oracle (Stellar Oracle Shield / Acurast) |
| Wallet | [Freighter](https://freighter.app) (`@stellar/freighter-api ^6`) |
| Frontend | Next.js 16, Tailwind CSS 4, Framer Motion |
| Network | Stellar Testnet → Mainnet |

---

## Get started

**Run the frontend:**

```bash
git clone https://github.com/LuckyPoolHQ/LuckyPool-frontend.git
cd LuckyPool-frontend
npm install
npm run dev
# → http://localhost:3000
```

Install the [Freighter](https://freighter.app) browser extension to connect a wallet.

**Build & test the contracts:**

```bash
git clone https://github.com/LuckyPoolHQ/LuckyPool-contracts.git
cd LuckyPool-contracts

make test                              # unit tests, no WASM needed
cargo install --locked stellar-cli
make build                             # build WASM

export SOURCE=alice ADMIN=G... USDC_ID=C... BLEND_ID=C... ORACLE_ID=C... FEE_BPS=500
make deploy && make initialize
```

Full contract interface and deploy guide: [LuckyPool-contracts README](https://github.com/LuckyPoolHQ/LuckyPool-contracts#readme).

---

<div align="center">

**No loss. Real yield. One lucky winner every week.**

</div>
