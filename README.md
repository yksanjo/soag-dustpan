# SOAG Dustpan

> A vacuum for your Solana wallet. Close empty SPL token accounts, reclaim the SOL rent, and convert it to $SOAG in one connect-and-sign flow.

**Live:** [agentsoag.com/dustpan](https://agentsoag.com/dustpan/)
**Mint:** [`ADue87cPcDhsyGq2hrDsukp7j8AFTSnaYHSanDATpump`](https://solscan.io/token/ADue87cPcDhsyGq2hrDsukp7j8AFTSnaYHSanDATpump)
**Treasury:** [`J7Vq2JjQoH72n1kzy1yjm4dCG65gw1oSVzMECwRyd7fP`](https://solscan.io/account/J7Vq2JjQoH72n1kzy1yjm4dCG65gw1oSVzMECwRyd7fP) (1% fee receiver)

---

## What it does

Every SPL token account on Solana requires **0.00203928 SOL** of rent as a storage deposit. When you trade a token to zero, that deposit stays locked in the empty account unless you explicitly close it. A typical pump.fun degen wallet has 50–500+ such empty accounts — sometimes more than 10,000 — meaning real money is pinned to dead memecoins.

Dustpan does three things, atomically per sweep:

1. **Closes** every empty SPL token account (TOKEN + TOKEN_2022 programs both supported)
2. **Transfers 1%** of the reclaimed SOL to the treasury wallet (visible on-chain)
3. **Swaps** the remaining 99% of reclaimed SOL → $SOAG via Jupiter, deposited to your wallet

All three happen under your wallet's signature. Dustpan never holds custody.

## How safe is it?

- **Non-custodial.** Your wallet signs every transaction. We hold no keys.
- **Open source.** This repo. Audit the entire mechanic.
- **Transparent fee.** The 1% transfer is a visible `SystemProgram.transfer` instruction in the same transaction. Your wallet shows you the destination before you sign.
- **Each tx is labelled.** Every batch carries an on-chain memo (`soag-dustpan/v1 sweep batch N/M`) so Solscan and Phantom can label the action.
- **Same close-ATA mechanic** as Sol-Incinerator / incinerator.fun — the difference is where the reclaimed SOL goes.

## Why $SOAG?

Dustpan is part of the [$SOAG](https://agentsoag.com) ecosystem — a Solana agent-tooling token with an active community. The 1% fee funds ecosystem development; the 99% Jupiter route creates organic buy pressure on the token. Every sweep is a small flywheel for everyone who holds.

## Architecture

Single-file vanilla JS + ESM imports from [esm.sh](https://esm.sh). No build step, no framework.

- `@solana/web3.js` — transaction construction
- `@solana/spl-token` — close-account instructions
- Helius RPC — token account scanning
- Jupiter v1 (`lite-api.jup.ag/swap/v1`) — SOL → $SOAG routing
- Memo program — instruction labeling

The whole app is in [`index.html`](./index.html) — ~1400 lines including the 8-bit phosphor-terminal UI.

## Limits

- **88 close instructions per sweep.** Wallets can choke on too many bundled transactions. Larger wallets sweep in multiple sessions.
- **Held (non-empty) dust isn't swapped yet.** v1 closes empties only. v2 will route Jupiter swaps for tradeable dust in the same flow.

## Run locally

Static file — open `index.html` in any browser. Requires a Solana wallet extension (Phantom, Solflare, Backpack). No build, no install.

```bash
git clone https://github.com/yksanjo/soag-dustpan.git
cd soag-dustpan
# open index.html
```

For local testing without committing to a sweep, change `MAX_CLOSE_PER_SWEEP` to a small number (e.g. `2`) at the top of the script tag.

## Config

Constants at the top of the `<script type="module">` block:

| Constant | Value | Purpose |
|---|---|---|
| `SOAG_MINT` | `ADue87cPc...pump` | Token to receive |
| `TREASURY` | `J7Vq2JjQ...d7fP` | 1% fee destination |
| `FEE_BPS` | `100` | Fee in basis points (1%) |
| `CLOSE_CHUNK` | `15` | Close ixs per transaction |
| `MAX_CLOSE_PER_SWEEP` | `88` | Soft cap per sweep |

## License

MIT — fork it, change the mint, use the close-and-route pattern for your own token. Just don't claim it's audited if you haven't actually audited it.

## Community

- [$SOAG Telegram](https://t.me/+ywGUPczH4GllYTBh)
- [@yksanjo](https://x.com/yksanjo) on X
- [agentsoag.com](https://agentsoag.com) — full $SOAG ecosystem
