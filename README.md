# ExitAnt Fibo R-R — Lighter

Deploy this automated trading bot to **your own Render account** as a background
worker. The bot runs from a prebuilt Docker image; this repository contains **only
the Render Blueprint (`render.yaml`)** — no source code.

## Deploy to Render

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/ExitAnt-dev/fibo-rr-lighter-3)

```
https://render.com/deploy?repo=https://github.com/ExitAnt-dev/fibo-rr-lighter-3
```

1. Click the button (or open the link) and sign in to Render.
2. Render reads `render.yaml` and proposes **one background worker in Frankfurt** with a
   **1 GB persistent disk** mounted at `/data` (settings, positions and the bot's trading key survive restarts and redeploys).
3. Enter the environment variables listed below. They are marked `sync: false`, so Render asks you for
   them on the deploy screen and stores them **only in your own Render account**.
4. Click **Apply** and wait for the first deploy to finish.
5. Open Telegram and send **`/start`** to your bot to begin setup.

## Before you deploy

- **Region: keep Frankfurt.** Lighter refuses requests from US IP addresses, so the Blueprint places the worker
  in Frankfurt. Do not move it to a US region.
- **A Lighter account with USDC in Perps.** Connect your wallet at [app.lighter.xyz](https://app.lighter.xyz) and
  deposit USDC — the first deposit creates the account. USDC in spot is not trading collateral.
- **Account type: Simple Trading Account.** The bot switches a Unified Trading Account to Simple by itself when
  no position is open (spot USDC must stay outside the collateral so the optional cash vault is kept apart).
- **Account tier: Plus.** The ExitAnt fee is only accepted on Plus accounts. If your account is on Standard, the bot
  asks for your consent in Telegram and switches it to Plus (Plus adds Lighter's own small trading fee).
- **ExitAnt fee approval.** At the first start the bot approves the ExitAnt integrator fee for your account
  (0.05% per trade) with your wallet signature. An existing higher approval is never lowered. This approval is the
  membership check; the bot does not trade until it is confirmed.
- **Trading key.** The bot creates its own Lighter API key (slot 251) with one wallet signature and keeps it on the
  `/data` disk. Other ExitAnt bots use other slots, so they can share the account — but do not run two bots on the
  **same symbol** of one account: Lighter keeps one position per market.

## Environment variables

| Variable | What to enter |
|---|---|
| `LIGHTER_WALLET_KEY` | The **private key** of the wallet you use on Lighter (`0x` + 64 hex characters) — not the address, not a recovery phrase, not a Lighter API key |
| `TELEGRAM_BOT_TOKEN` | Bot token issued by [@BotFather](https://t.me/BotFather) |
| `TELEGRAM_CHAT_ID` | Your own Telegram chat ID — the bot ignores every other sender and every group |

Nothing else to configure: the exchange, trading mode and state-file location are built into the image.

## What the bot does

Fibo R-R waits for a Bollinger Band breakout on the 5- or 15-minute chart, then enters in up to six steps
(levels) at the spacing you configure, takes profit per level at a Fibonacci ratio towards the opposite
band breakout, keeps one exchange stop-loss for the whole position at your loss limit, and resets the
ladder on an opposite breakout, after a time limit or by hand. A margin guard trims the position when used
margin exceeds your threshold. Every parameter is set in Telegram; twelve languages are supported.
On Lighter a symbol runs long **or** short (one position per market), and amounts are in USDC.

## How it works

- **Prebuilt image.** The worker pulls `docker.io/exitant/exitant-fibo-rr-lighter:3` from Docker Hub. The image ships
  a compiled binary only — no Python source is included or exposed.
- **Background worker.** There is no web page; the bot runs continuously and talks to you through Telegram.
- **Your key stays yours.** The wallet key is read from your own Render environment, is used only to sign two
  transactions (the trading-key registration and the fee approval) and is never logged, stored or sent anywhere.
  The bot never asks for a key in Telegram — if you paste one into the chat, it deletes the message and warns you.
- **Persistent state.** Settings, positions, pending orders and the bot's trading key live in a small database on the `/data` disk.

## Important

- **Keep the instance count at 1.** Two instances cause a Telegram `getUpdates` conflict.
- Render **background workers have no free plan** — choose Starter or higher.
- **The bot trades live from the first start.** Begin with a small allocation and watch one full cycle
  (breakout → entries → take-profit/stop-loss) on your account before scaling up.
- The smallest order on Lighter is about 10 USDC; the setup only offers sizes that reach it.
- After the deploy succeeds, send **`/start`** in Telegram to configure the bot.

---

Blueprint spec: <https://render.com/docs/blueprint-spec> · Prebuilt images: <https://render.com/docs/deploying-an-image>
