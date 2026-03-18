# SSCN — Structured Security Crypto Note

**A fully on-chain tokenized Reverse Convertible Note on ETH/USD**  
ERC-3643 (T-REX) · Sepolia Testnet · Chainlink Oracle · OnchainID

> Academic project — Master Finance, Technology & Data  
> Paris 1 Panthéon-Sorbonne · 2026  
> BOUZIANI Shéhérazade · SYLLA Mourad

---

## What is SSCN?

SSCN is a tokenized structured product — a **Reverse Convertible Note** on ETH/USD — deployed on the Ethereum Sepolia testnet. It replicates the payoff structure of a real capital markets instrument using smart contracts, a live Chainlink price oracle, and a compliant identity framework based on ERC-3643 (T-REX).

### Payoff structure

At maturity, the investor receives:

- **If ETH/USD ≥ Barrier at maturity (no breach):** full capital ($1,000/token) + coupon (10%)  → **+$100 (+10%)**
- **If ETH/USD < Barrier at maturity (breach):** capital is reduced proportionally to ETH performance + coupon is still paid  → **loss up to −40% or more depending on ETH drop**

The coupon (put premium) is pre-funded by the issuer before token issuance, guaranteeing the investor always receives the coupon regardless of the barrier outcome.

---

## Live demo

**dApp:** [she8075.github.io/SSCN](https://she8075.github.io/SSCN/)

| Role | Description |
|------|-------------|
| **Issuer** | Deploy contract, monitor subscriptions, deposit coupon, issue tokens, settle at maturity |
| **Investor** | Review term sheet, subscribe with USDC, track investment, redeem payoff |

---

## Architecture

### Smart contracts (Sepolia)

| Contract | Address | Role |
|----------|---------|------|
| `SSCNToken_RC_v4.sol` | `0x872fB88bE0b7fCa437D43143D465F44A62c5c9e4` | Reference SSCN contract (MockKYC) |
| `SSCNToken_RC_v4.sol` | `0x735188235B28E7A28ae37A9B65F99cd5D7d4D0FA` | SSCN with real OnchainID registry |
| `ClaimIssuer.sol` | `0xF5688F0d72eFa3542fb272724B1F10889A176761` | KYC claim issuer (ERC-735) |
| `Identity.sol` | `0xE651deA46a36311bbf4ede90CA6b4069D8d26216` | Investor identity contract (ERC-734) |
| `IdentityRegistry.sol` | `0xC6Dc128CfA93bdE19aFd8860770a8C8eA68875B6` | T-REX identity registry |
| `MockKYC` | `0x1DDd83f21D764369807a789CFFe60a651E011aF4` | Simplified KYC stub (demo only) |
| `MockUSDC` | `0x19D76090915c4f97fF2075C3AC0822AbE1898E91` | ERC-20 test stablecoin |
| Chainlink ETH/USD | `0x694AA1769357215DE4FAC081bf1f309aDC325306` | Live price feed (Sepolia) |

### SSCNToken_RC_v4.sol — key functions

| Function | Role | Caller |
|----------|------|--------|
| `subscribeAndDeposit(uint256)` | Investor subscribes and locks USDC atomically | Investor |
| `unsubscribe()` | Investor exits during subscription period | Investor |
| `depositCoupon()` | Issuer pre-funds the coupon | Issuer |
| `closeSubscription()` | Closes the subscription window | Issuer |
| `issueAll()` | KYC-checks all subscribers and mints tokens | Issuer |
| `settleAtMaturity()` | Reads Chainlink price, computes payoff, transitions to SETTLED | Issuer |
| `redeem()` | Investor receives capital + coupon | Investor |

### Contract lifecycle

```
SUBSCRIPTION → ACTIVE → SETTLED
```

**SUBSCRIPTION:** investors can subscribe and deposit USDC. Issuer deposits coupon.  
**ACTIVE:** subscription closed, tokens issued to KYC-verified investors.  
**SETTLED:** `settleAtMaturity()` called after maturity, payoff computed from Chainlink ETH/USD.

---

## ERC-3643 (T-REX) compliance

SSCN implements the core identity verification architecture of the ERC-3643 standard, which governs compliant security token transfers.

### What was implemented

**ClaimIssuer** (`0xF568...761`) — the issuer wallet acts as the KYC agent. It can issue and revoke claims about investor identities. Implements `isClaimValid()` using ECDSA signature verification.

**Identity contract** (`0xE651...216`) — each investor has their own on-chain identity smart contract compliant with ERC-734/735. It stores claims issued by authorized claim issuers.

**IdentityRegistry** (`0xC6Dc...B6`) — the central registry linking investor wallet addresses to their Identity contracts. The SSCN contract calls `isVerified(investor)` on this registry before allowing subscription or token issuance.

With this setup, **investors registered in the IdentityRegistry can subscribe to any new SSCN contract without any manual KYC action from the issuer** — verification is enforced automatically on-chain.

### Architectural scope

The following components of the full Tokeny T-REX implementation were intentionally excluded as they do not add demonstrable value in an academic testnet context:

- `ClaimTopicsRegistry` — centralizes required claim types per token
- `TrustedIssuersRegistry` — whitelist of authorized claim issuers per topic
- `Compliance` module — encodes transfer restrictions (jurisdiction caps, max holders, lockup)
- `IdFactory` with beacon proxy — deterministic cross-chain Identity deployment via CREATE2

The identity contracts were written from scratch without copying the Tokeny open-source codebase.

---

## dApp — frontend

**Files:**
- `index.html` — main dApp (Issuer + Investor tabs)
- `index_sandbox.html` — sandbox mode with MockPriceFeed for barrier breach simulation

**Stack:** vanilla HTML/CSS/JS · ethers.js v6 · QRCode.js · Chart.js  
No build step. Deployed as a static site via GitHub Pages.

### Dual-wallet architecture

The dApp manages two independent wallet connections simultaneously:
- **Issuer wallet** (`0x53a8...`) — connected in the Issuer tab
- **Investor wallet** (`0xDa4f...`) — connected in the Investor tab

Both wallets can be connected at the same time in different browser tabs, simulating a real-world issuer/investor interaction.

### Key UX features

- **Live Chainlink price chart** — ETH/USD with barrier overlay (last 24h via Deribit API)
- **Term sheet modal** — displays full product terms before subscription, checks USDC balance, offers testnet mint if insufficient
- **QR code sharing** — issuer generates a QR code that deep-links the investor directly to the right contract
- **Subscription monitoring** — real-time list of subscribers with subscription timestamp and USDC locked
- **Settlement result panel** — displays S(T), barrier status, capital per token, coupon, total payoff, P&L
- **Sandbox tab** — deploy a `MockPriceFeed` to control S₀ and S(T) manually, test both barrier scenarios

---

## How to run a demo

### Standard flow (Chainlink oracle, MockKYC)

1. Open [she8075.github.io/SSCN](https://she8075.github.io/SSCN/) — **Issuer tab**
2. Connect issuer wallet (MetaMask, Sepolia)
3. Section 02 — set parameters, leave Identity Registry blank (uses MockKYC)
4. Click **Deploy Contract**
5. Step 2 — click **Generate QR / Link** and share with investor
6. Open a second browser or tab — **Investor tab** — connect investor wallet
7. Load contract via QR or "From Issuer" button
8. Step 2 (Investor) — click **Subscribe**, accept term sheet, mint test USDC if needed
9. Step 3 (Issuer) — monitor subscriptions, verify investor KYC manually
10. Step 4 — **Approve USDC → Deposit Coupon**
11. Step 5 — **Issue All Tokens**, wait for maturity, **Settle at Maturity**
12. Investor — **Redeem** (button activates automatically after settlement)

### T-REX flow (real OnchainID, no manual KYC)

Same as above but in step 3, paste the deployed IdentityRegistry address:
```
0xC6Dc128CfA93bdE19aFd8860770a8C8eA68875B6
```
The investor (`0xDa4fe98782629879f15DE43a922D9079410E6C50`) is permanently registered and can subscribe without any issuer KYC action.

### Sandbox flow (manual price control)

1. Open `index_sandbox.html`
2. Section 00 — **Deploy MockPriceFeed** with S₀ of your choice
3. Section 01 — **Deploy SSCN** using the MockPriceFeed address
4. Run the full subscription flow
5. Before settling — go back to Section 00, set S(T) to any value (e.g. $800 = −60% → barrier breached)
6. **Settle at Maturity** — observe capital loss in settlement result

---

## On-chain financing protection

The contract includes a protection against under-funded issuance. If the issuer attempts to issue more tokens than the deposited USDC can cover (coupon not fully funded), the transaction reverts with:

```
SSCN: insufficient funds to cover issuance
```

**Test validated:** contract `0xFEf1047Fc409069EcB340A6FcB5418C6c17e1F9F`, deposit of $1,100 (1 token), attempted issuance of 3 tokens → revert ✅

---

## Limitations and known constraints

**MockUSDC:** the testnet uses a mintable ERC-20 stub. In production, investors would hold real USDC — the "Get test USDC" button does not exist in a real deployment.

**Sepolia RPC reliability:** public Sepolia RPC nodes (`publicnode.com`) occasionally fail `eth_estimateGas` on complex contracts. All transactions use explicit `gasLimit` values to bypass estimation.

**OnchainID scope:** the T-REX implementation covers the identity verification core (ClaimIssuer, Identity, IdentityRegistry) but not the full Tokeny suite (ClaimTopicsRegistry, TrustedIssuersRegistry, Compliance module, IdFactory beacon proxy).

**Deployment restriction:** in the demo dApp, the Deploy button is accessible to any connected wallet. In production, deployment would be restricted to the authorized issuer via authentication middleware.

**Barrier type:** European only (checked at maturity). American (daily barrier monitoring) is not implemented.

**Single investor per address:** each wallet can subscribe once per contract. Multiple tokens per subscription are supported via the `amount` parameter.

---

## Project context

This project was developed as **Stage 4** of a structured product tokenization curriculum as part of the Master Finance, Technology & Data program at Paris 1 Panthéon-Sorbonne.

Previous stages:
- **Stage 1–2:** product design and Black-Scholes pricing model
- **Stage 3:** backtesting GBM model assumptions against real ETH/USDT historical data (Binance API, 2021–2026), including regime-conditional analysis and block bootstrap ESS estimation
- **Stage 4:** smart contract development, ERC-3643 compliance layer, and full-stack dApp

---

## License

Academic project — Paris 1 Panthéon-Sorbonne · 2026
