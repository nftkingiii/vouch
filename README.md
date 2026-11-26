# Vouch

A skill for OpenClaw and Hermes agents that checks a consequential claim before the agent acts on it. It builds on Telegraph's own MCP server (`@telegraphprotocol/telegraph-protocol-mcp`) instead of replacing it.

For each claim the agent finds the right Telegraph intent, asks independently ranked miners within a stated budget, compares the answer with the claim, and reports **verified**, **contradicted** or **cannot verify**, each with the Telegraph receipt and the cost. A failed, empty or irrelevant answer is never treated as confirmation. An incomplete answer gets one recovery ask to a second ranked miner. Caps default to $0.05 per claim and $0.25 per task, and a ledger keeps a running tally.

Telegraph commercial use case #20: independent agent task verification.

Status: built and tested against the Telegraph testnet with a real escrow-paid MCP server. Not published anywhere. See `demo/RESULTS.md` for what was run and what was not (no real OpenClaw or Hermes agent session).

## Layout

```
skills/vouch/
  SKILL.md                  the procedure (agentskills.io format; works in OpenClaw and Hermes)
  scripts/vouch.mjs         dependency-free ledger, caps, tolerance judge, report
  references/intent-map.md  claim to intent starting points
  references/answer-format.md
docs/openclaw.md            MCP config + skill install for OpenClaw
docs/hermes.md              MCP config + skill install via a Hermes tap
demo/                       transcripts, ledgers and receipts from the testnet runs
```

The `skills/<name>/SKILL.md` layout is also what a Hermes tap expects.

## Install

- OpenClaw: [docs/openclaw.md](docs/openclaw.md)
- Hermes: [docs/hermes.md](docs/hermes.md)

No key is shipped. You bring your own Telegraph wallet and escrow (or Anjoman API key), on the testnet.

## Helper

```
node skills/vouch/scripts/vouch.mjs init --task "name" --claim-cap 0.05 --task-cap 0.25
node skills/vouch/scripts/vouch.mjs claim --id c1 --text "..." --tolerance "..."
node skills/vouch/scripts/vouch.mjs check --claim c1 --est 0.01      # exit 3 = refused, over cap
node skills/vouch/scripts/vouch.mjs record --claim c1 --intent X --miner M --cost 0.01 --receipt 0x...
node skills/vouch/scripts/vouch.mjs judge --claimed 5 --observed 0.045 --tol lt
node skills/vouch/scripts/vouch.mjs verdict --claim c1 --verdict verified --observed "..." --why "..."
node skills/vouch/scripts/vouch.mjs report
```

`verdict` refuses verified or contradicted when no receipt is recorded for the claim.

## License

MIT
