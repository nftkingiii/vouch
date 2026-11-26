# Vouch on OpenClaw

Vouch is a skill that sits on top of Telegraph's own MCP server. You add the server once, then the skill.

Everything here points at the Telegraph testnet (Ethereum Sepolia, test USDC). Use a wallet made only for this. No key goes into any file in this repo, into OpenClaw's config, or into chat.

## 1. Telegraph wallet and escrow (once, in your own terminal)

```bash
npx -y @telegraphprotocol/telegraph-protocol-mcp@2.3.0 setup
```

Setup creates a wallet at `~/.telegraph/wallet.json` (mode 0600), shows the address, and moves the USDC you choose into escrow after you confirm. Test USDC comes from https://anjoman.ai/dashboard/faucet; gas needs a little Sepolia ETH from a public faucet. Check later with `npx -y @telegraphprotocol/telegraph-protocol-mcp@2.3.0 status`.

If you have an Anjoman API key instead of a wallet, set `TELEGRAPH_API_KEY` in the server's `env` (step 2) instead. Keep it out of the repo.

## 2. Register the MCP server, with spend caps

```bash
openclaw mcp add telegraph \
  --command npx \
  --arg=-y --arg @telegraphprotocol/telegraph-protocol-mcp@2.3.0 \
  --env TELEGRAPH_MAX_ASK_USD=0.05 \
  --env TELEGRAPH_DAILY_LIMIT_USD=0.25 \
  --env TELEGRAPH_MAX_BUDGET_USDC=50000
openclaw mcp doctor telegraph --probe
```

If your `openclaw mcp add` does not take `--env`, merge this into `~/.openclaw/openclaw.json` instead:

```json
{
  "mcp": {
    "servers": {
      "telegraph": {
        "command": "npx",
        "args": ["-y", "@telegraphprotocol/telegraph-protocol-mcp@2.3.0"],
        "env": {
          "TELEGRAPH_MAX_ASK_USD": "0.05",
          "TELEGRAPH_DAILY_LIMIT_USD": "0.25",
          "TELEGRAPH_MAX_BUDGET_USDC": "50000"
        }
      }
    }
  }
}
```

The three variables are the server-side backstop for Vouch's budget: $0.05 per ask, $0.25 per UTC day, and a $0.05 multi-miner budget (50000 micro-USDC) per `telegraph_ask`. They hold even if the model ignores the skill. The server defaults are $0.50 per ask and $5 per day.

## 3. Install the skill

Once Vouch is on ClawHub:

```bash
clawhub install vouch
```

Or from a checkout of this repo, into your personal skills directory:

```bash
cp -r skills/vouch ~/.openclaw/skills/vouch
```

The skill is gated on `mcp.servers.telegraph` (`metadata.openclaw.requires.config`) and on `node` being on `PATH`, so it stays hidden until step 2 is done. Start a new session so it is picked up.

## 4. Check it

Ask the agent:

> Before you act on it, verify this with Vouch: Ethereum gas is below 5 gwei, and the Golden Knights beat the Panthers 9-3 on 2023-06-13. Budget $0.10.

You should see free `telegraph_*` lookups, one or two $0.01 asks, and a per-claim verdict with receipts and cost.

## Notes

- The helper `scripts/vouch.mjs` writes its tally to `~/.vouch/ledger.json` (override with `VOUCH_LEDGER`). It holds no secrets.
- OpenClaw may prefix MCP tool names; the skill refers to them by their Telegraph names.
- Funding, deposits and withdrawals stay in your terminal. The skill tells the agent never to call `telegraph_deposit`, run `fund`, or read the wallet file. An agent with shell access could still read the wallet file, so keep only test funds in it.
