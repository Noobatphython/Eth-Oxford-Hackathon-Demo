# Eth-Oxford-Hackathon-Demo

# GasCap Futures

A cash-settled gas price futures exchange built on Flare Network. Trade long or short on a crypto gas price index using live FTSO oracle feeds.

Built at ETH Oxford 2026 by Aron(me),(and teammates) 

<img width="1919" height="918" alt="image" src="https://github.com/user-attachments/assets/de189be5-64d1-484b-9f9c-c3511e282397" />

Try it out (after reading below)


## What is this?

There's no way to hedge gas price risk in crypto. Gas costs spike unpredictably and you just have to absorb it. GasCap lets you trade futures on a gas price index so you can actually manage that exposure.

The index is a weighted average pulled from Flare's FTSO v2 oracle: BTC (50%) + ETH (30%) + FLR (20%), updated roughly every 1.8 seconds. The chart shows real oracle prices — no fake data after the initial page load.

Instead of an order book matching buyers and sellers, we use a **liquidity pool**. LPs deposit funds, the pool takes the other side of every trade, and at expiry the smart contract reads the FTSO price and settles everything automatically. No middleman, no clearinghouse.

## How it works

1. A market gets created with a **strike price** and **expiry time**
2. LPs add liquidity to the pool, this funds settlement payouts 
3. Traders open **long** (gas goes up) or **short** (gas goes down) positions, posting collateral in C2FLR
4. At expiry, the contract locks the oracle price as the settlement price
5. Payout = `(settlementPrice - strikePrice) × quantity × leverage + collateral`
6. Traders claim their payout on the Settle page. LPs get the rest plus fees

Everything is on-chain. Settlement is trustless — the contract reads the FTSO feed and pays out accordingly.

## Features

- **Live FTSO price feed** — real oracle data plotted on a TradingView-style candlestick chart (1m, 5m, 15m, 60m timeframes)
- **Long and short positions** — bet on gas going up or down
- **Leverage** — 1x to 100x (use carefully, there's no liquidation engine)
- **Liquidity pool** — LPs provide capital and earn from trading fees
- **Automatic settlement** — contract reads oracle at expiry, calculates payouts, done
- **Factory pattern** — anyone can create new markets with custom strike prices and expiry times
- **On-chain transparency** — every trade, deposit, and settlement is visible on the Coston2 block explorer
- **Depeg Shield (Phase 2)** — stablecoin depeg protection using FTSO + FDC cross-chain verification (separate branch)

## Try the demo

**Live demo:** [eth-oxford-future-863727-5575b.web.app](https://eth-oxford-future-863727-5575b.web.app)

### What you need

- **MetaMask** (or any EVM wallet)
- **Coston2 testnet C2FLR tokens** — free test tokens, not real money. Get them from the [Coston2 faucet](https://faucet.flare.network/coston2)

### Setup

Add Coston2 to MetaMask (the app should prompt you automatically, but if not):

| Setting | Value |
|---------|-------|
| Network Name | Coston2 |
| RPC URL | `https://coston2-api.flare.network/ext/C/rpc` |
| Chain ID | 114 |
| Symbol | C2FLR |
| Explorer | `https://coston2-explorer.flare.network` |

Get test C2FLR from the faucet, connect your wallet, and you're in.

### Trading

1. Pick a market from the dropdown at the top
2. Choose **LONG** or **SHORT** on the right panel
3. Set your quantity (contracts) and collateral (C2FLR)
4. Submit through MetaMask
5. Wait for expiry — the 48h market expires Feb 10, 00:41 UTC as an example
6. Go to the **Settle** page to claim your payout after settlement

### Adding liquidity

1. Go to the **Pool** page
2. Deposit C2FLR into the pool
3. Your share of the pool earns fees from trading activity

## Caveats and honest limitations

This was built in 43 hours at a hackathon. It works, but here's what you should know:

**The gas index is an estimate.** The "gas price" is derived from FTSO price feeds (BTC/ETH/FLR weighted), not actual Ethereum network gas costs. We started integrating real gas data via Flare's FDC protocol and Beaconcha.in, but the FTSO-derived index is what's live. The numbers on the chart are directionally right but not exact gas prices.

**The pool has no money in it.** This is a testnet demo — nobody has deposited meaningful liquidity. To actually test a full trade cycle, you'd need to:
1. Add liquidity yourself on the Pool page (be both the LP and the trader)
2. Open a position
3. Wait for expiry
4. Settle and claim

**The 48h expiry is just an example.** We set one market to 30 seconds (for quick demo purposes) and one to 48 hours (to show a realistic timeframe). You can create markets with any expiry via the Factory contract.

**No liquidation engine.** Leveraged positions can go underwater and there's nothing to close them early. In a real product you'd need this — we didn't have time.

**One position per trader per market.** Simplification for the MVP. A real exchange would let you have multiple.

**The frontend seeds some initial candles** so the chart doesn't look empty on first load. After that, all price data is live from FTSO.

## On-chain activity

Everything is verifiable on the [Coston2 block explorer](https://coston2-explorer.flare.network). You can see our example transactions : market creation, liquidity deposits, position opens — all public on-chain.

| Contract | Address |
|----------|---------|
| Factory | [`0x04932e0F...be465Adb`](https://coston2-explorer.flare.network/address/0x04932e0F...be465Adb) |
| Main Market | [`0xCeBEbB73...FdE04d5d`](https://coston2-explorer.flare.network/address/0xCeBEbB73...FdE04d5d) |
| Market (30s) | [`0x5E4BfBBb...DA001A63`](https://coston2-explorer.flare.network/address/0x5E4BfBBb...DA001A63) |
| Market (48h) | [`0x6c388702...Eff406771`](https://coston2-explorer.flare.network/address/0x6c388702...Eff406771) |

## Economic design

**Why a pool instead of an order book?**

Futures need a counterparty for every trade. On a testnet with no users, there's nobody on the other side. The LP pool solves this — it always takes the other side, so you can trade immediately even with zero other traders.

**Reserve factor (70%)** limits how much exposure the pool can take relative to its liquidity. If the pool has 100 C2FLR, max total exposure is 70 C2FLR. This prevents the pool from being unable to cover payouts if all positions go against it.

**How this could be improved:**
- Dynamic fee adjustment based on pool utilization and position skew
- Proper liquidation engine for leveraged positions
- ERC20 token support (currently native C2FLR only)
- Real gas data via FDC attestation instead of derived index
- Multiple positions per trader
- LP tokenisation (receipt tokens for pool shares)

## Tech stack

| Layer | Tech |
|-------|------|
| Smart contracts | Solidity 0.8.19, Hardhat |
| Network | Flare Coston2 testnet (Chain 114) |
| Oracle | Flare FTSO v2 (TestFtsoV2Interface) |
| Frontend | Next.js, React, ethers.js |
| Charts | lightweight-charts (TradingView) |
| UI | Tailwind CSS, shadcn/ui, dark terminal theme |

## Also built: Depeg Shield (Phase 2)

Stablecoin depeg protection — parametric insurance where you pay a premium and get paid out automatically if USDC or USDT drops below a price barrier for 15+ minutes. Uses FTSO for price monitoring and FDC for cross-chain verification that the depeg is real and systemic, not just one exchange glitching. On a separate branch, not fully integrated into the main app yet.

## License

MIT
