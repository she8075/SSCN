# SSCN — Structured Security Crypto Note

> A fully on-chain tokenized structured product — Reverse Convertible on ETH/USD  
> ERC-3643 (T-REX) · Chainlink Oracle · Sepolia Testnet

**🌐 Live dApp → [she8075.github.io/SSCN](https://she8075.github.io/SSCN)**

---

## What is SSCN?

SSCN is a tokenized Reverse Convertible structured product built on Ethereum. It offers investors a **guaranteed 10% annual coupon** with capital exposure to ETH/USD via a European barrier option mechanism.

| Parameter | Value |
|-----------|-------|
| Underlying | ETH/USD (Chainlink oracle) |
| Barrier type | European — observed **once** at maturity |
| Barrier level | 60% × S₀ |
| Coupon | 10% / year — **always guaranteed** |
| Nominal | $1,000 per token |
| Settlement | MockUSDC (Sepolia testnet) |
| Standard | ERC-3643 (T-REX) |

---

## Payoff Structure

```
If S(T) ≥ 60% × S₀  →  $1,000 capital + $100 coupon  =  $1,100
If S(T) <  60% × S₀  →  $1,000 × S(T)/S₀ + $100 coupon
```

The coupon is **always paid**, regardless of barrier breach. Only the capital is at risk.

---

## Architecture

```
SSCNToken_RC.sol          — Main ERC-3643 Reverse Convertible contract
MockUSDC.sol              — Test USDC (6 decimals)
MockKYC.sol               — KYC identity registry (IIdentityRegistry interface)
SSCN_ONCHAINID_Integration.sol  — ONCHAINID compliance architecture
SSCN_Backtesting.py       — Historical backtesting (Binance API, 825 windows)
SSCNToken_RC_t.sol        — Foundry test suite (62/62 PASS)
index.html                — Interactive dApp (this site)
```

---

## Deployed Contracts — Sepolia Testnet

| Contract | Address | Description |
|----------|---------|-------------|
| MockKYC | `0x1DDd83f21D764369807a789CFFe60a651E011aF4` | KYC registry |
| MockUSDC | `0x19D76090915c4f97fF2075C3AC0822AbE1898E91` | Test USDC |
| SSCNToken_RC (Sc.1 — No breach) | `0xA02B28c45f814B7FF6c663cF1db5aFBa86edb571` | barrierPct=6000, settled ✅ |
| SSCNToken_RC (Sc.2 — Breach test) | `0xe7ffc97766e6b198938e50857902d5012069C5dC` | barrierPct=9900, settled ✅ |
| MockONCHAINID | `0x42fB8dc936dd5958514CE78Fe6D853aa9eC00855` | ONCHAINID mock |
| SSCNComplianceChecker | `0xF2CD3636caC2784B629d97A668c3871C3d906a8a` | Compliance layer |
| SSCNIdentityRegistryAdapter | `0x8E75e887A32A24614C5d1619c3144684a0269958` | Identity adapter |

---

## Backtesting Results

Historical analysis on **825 rolling 1-year windows** of ETH/USD (Binance, 2022–2025):

| Metric | Value |
|--------|-------|
| P(barrier breach at maturity) | **11.8%** |
| Expected payoff | **$1,044** |
| Median payoff | **$1,100** |
| VaR 95% | **$627** |
| CVaR 95% | **$565** |
| Avg realized volatility | **~53%** |

### Stress Event Analysis

| Event | Max drawdown | Barrier breached | Worst-case payoff |
|-------|-------------|-----------------|-------------------|
| LUNA/UST crash (Apr–May 2022) | −71.7% | Yes (American) / No (European) | $383 |
| FTX collapse (Nov 2022) | −44.4% | No (European — recovered at maturity) | $1,100 |

> The European barrier is central to the SSCN design: despite a −44.4% intraday drawdown during the FTX crisis, ETH recovered fully by maturity → $1,100 payoff.

---

## Test Suite — Foundry

```
62/62 tests PASS
├── Unit tests: deposit, issue, settle, redeem
├── Scenario 1: no breach → $1,100 payoff
├── Scenario 2: breach → reduced capital + guaranteed coupon
├── KYC enforcement: revert on unverified investor
├── State machine: ACTIVE → SETTLED
└── Fuzz tests (4): arbitrary prices, barrier percentages
```

---

## How to Use the dApp

### Desktop
1. Install [MetaMask](https://metamask.io)
2. Switch to **Sepolia testnet**
3. Go to [she8075.github.io/SSCN](https://she8075.github.io/SSCN)
4. Click **Connect Wallet**

### Mobile
1. Open the **MetaMask app**
2. Tap **Explorer** (bottom tab)
3. Navigate to `she8075.github.io/SSCN`
4. Click **Connect Wallet**

### Get Sepolia ETH (for gas)
→ [sepoliafaucet.com](https://sepoliafaucet.com) or [faucet.sepolia.dev](https://faucet.sepolia.dev)

---

## Deploy Your Own Contract

From the dApp → **Deploy** section, choose your parameters:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `barrierPct` | Barrier as % × 100 | `6000` = 60% |
| `couponBps` | Annual coupon in basis points | `1000` = 10% |
| `maturityDelay` | Time to maturity in seconds | `604800` = 7 days |

Presets available: 5min test · 15min test · 1h test · 7d · 1 year

---

## ONCHAINID Integration

The SSCN compliance architecture follows the ERC-3643 / T-REX standard:

- **MockONCHAINID** deployed on Sepolia (replaces full Tokeny T-REX stack for POC)
- **SSCNComplianceChecker** — on-chain KYC verification
- **SSCNIdentityRegistryAdapter** — zero-redeployment migration path documented
- 8 transactions validated on-chain (blocks 10410823–10410901)

Production migration path to real ONCHAINID (Tokeny, Sum&Substance) documented in Stage 3 report.

---

## Authors

**BOUZIANI Shéhérazade** — Finance & Structuring  
**SYLLA Mourad** — Blockchain & Smart Contracts  

MSc Finance, Technology & Data — Paris 1 Panthéon-Sorbonne · March 2026

---

## Links

- 🌐 [Live dApp](https://she8075.github.io/SSCN)
- 🔍 [Etherscan — MockKYC](https://sepolia.etherscan.io/address/0x1DDd83f21D764369807a789CFFe60a651E011aF4)
- 🔍 [Etherscan — MockUSDC](https://sepolia.etherscan.io/address/0x19D76090915c4f97fF2075C3AC0822AbE1898E91)
- 🔗 [Chainlink ETH/USD Sepolia](https://sepolia.etherscan.io/address/0x694AA1769357215DE4FAC081bf1f309aDC325306)
