# Vouch on Hermes Agent

Hermes loads the skill from a GitHub tap and talks to Telegraph through its MCP support.

Everything here points at the Telegraph testnet (Ethereum Sepolia, test USDC). Use a wallet made only for this. No key goes into config files in this repo or into chat.

## 1. Telegraph wallet and escrow (once, in your own terminal)

```bash
npx -y @telegraphprotocol/telegraph-protocol-mcp@2.3.0 setup
```

This creates `~/.telegraph/wallet.json` (mode 0600) and funds escrow after you confirm. Test USDC: https://anjoman.ai/dashboard/faucet. Gas: a little Sepolia ETH from a public faucet. Node 20 or later must be on `PATH`, because Hermes starts the server with `npx`.

## 2. Add the MCP server to `~/.hermes/config.yaml`

```yaml
mcp_servers:
  telegraph:
    command: "npx"
    args: ["-y", "@telegraphprotocol/telegraph-protocol-mcp@2.3.0"]
    env:
      TELEGRAPH_MAX_ASK_USD: "0.05"
      TELEGRAPH_DAILY_LIMIT_USD: "0.25"
      TELEGRAPH_MAX_BUDGET_USDC: "50000"
```

Hermes does not pass your shell environment to stdio servers, only the `env` you list plus a safe baseline. The server finds its wallet through `~/.telegraph/wallet.json`, so no key is listed. If `HOME` is not carried through in your setup, add `TELEGRAPH_HOME: "/home/you/.telegraph"` to `env`. Tools appear as `mcp_telegraph_telegraph_find_intents`, `mcp_telegraph_telegraph_ask` and so on. Reload with `/reload-mcp` or restart Hermes.

The three variables are Vouch's server-side backstop: $0.05 per ask, $0.25 per UTC day, and a $0.05 multi-miner budget per `telegraph_ask`.

Note that Hermes' own docs say that in a non-interactive run, any call that needs approval is refused. Vouch's asks are paid by the MCP server, so check how your Hermes setup treats paid MCP tools before a headless run.

## 3. Install the skill from a tap

After the Vouch repo is public on GitHub (replace `OWNER`):

```bash
hermes skills tap add OWNER/vouch
hermes skills install OWNER/vouch/vouch
```

The repo layout is what Hermes expects: `skills/vouch/SKILL.md` plus `scripts/` and `references/`. A new tap gets `community` trust, so the install runs Hermes' security scan and shows a third-party warning. Vouch ships one small Node script, `scripts/vouch.mjs`, which makes no network calls and reads no secrets. You can also install straight from the repo without adding the tap (the second command alone).

## 4. Check it

> Use the vouch skill: verify that Ethereum gas is below 5 gwei and that the Golden Knights beat the Panthers 9-3 on 2023-06-13. Budget $0.10 in total.

Expect free lookups, one or two $0.01 asks, and a verdict per claim with receipts and cost.

## Notes

- The ledger lives at `~/.vouch/ledger.json` (override with `VOUCH_LEDGER`).
- The skill's frontmatter carries `metadata.hermes` tags. It does not require specific Hermes toolsets; if no Telegraph tool is loaded it tells the agent to stop rather than guess.
