# Demo results (Telegraph testnet, 2026-11-26)

How it was run: `@telegraphprotocol/telegraph-protocol-mcp@2.3.0` installed in `~/vouch` on the shared testbench VM, escrow mode, the Reckon testnet burner (key never printed), server caps `TELEGRAPH_MAX_ASK_USD=0.05`, `TELEGRAPH_DAILY_LIMIT_USD=0.25`, `TELEGRAPH_MAX_BUDGET_USDC=50000`. A small Node MCP client drove the real server over stdio. Judgements and verdicts were made by the Claude Code session following `SKILL.md` step by step, using `scripts/vouch.mjs` for the ledger, caps and tolerance checks. **No OpenClaw or Hermes agent session was run** (no LLM provider was configured for them on the VM, and none was provided).

Total paid: 9 asks, $0.09, all billed through escrow (`telegraph_account` reported `spent_today_usd: 0.09`).

| Task | Claim | Verdict | Evidence | Asks / cost |
|---|---|---|---|---|
| A c1 | Ethereum gas below 5 gwei | verified | LiveCert 0.0511 gwei, but with "conflicting information for gas_price_gwei": not accepted alone. One recovery ask to a1-preflight (rank 2): 0.0453 gwei, clean. Both far below 5. | 2 / $0.02 |
| A c2 | Golden Knights beat Panthers 9-3, 2023-06-13 | verified | TxLens: Vegas 9, Florida 3, 2023-06-13, all fields ok | 1 / $0.01 |
| B c1 | Same game, altered to 5-3 | contradicted | TxLens: 9-3 for the same game | 1 / $0.01 |
| B c2 | LINK total supply is 500,000,000 | contradicted | LiveCert: 1,000,000,000 (outside 1%) | 1 / $0.01 |
| C c1 | Oilers beat Flames 4-2 on 2027-03-14 (future) | cannot verify | TxLens returned "Real Madrid 2-1 Barcelona" for that date, all fields marked ok, with a conflicting-sources warning: wrong entity and a future date. Recovery ask to a1-preflight failed (500, upstream 402), not billed. | 2 asks, 1 billed / $0.01 |
| C c2 | Bitcoin above $200k on 2027-01-31 (forecast) | cannot verify | No intent can establish a forecast; nothing bought | 0 / $0 |

Receipts (escrow rail, epoch 59 or 60):

- A c1: `0x63081ddd843f29554c04dc464de1c9ed45e5a2795874e33fb3808c6e2b8a0fa1` (LiveCert), `0xa07f6e8494b7811e2f8a7dbc09fb5f1a33cd1625fd78e3f36a5588d89414aecd` (PREFLIGHT)
- A c2: `0x5add09ebb081ae1922ededa763d2f22fcf954ff4981ca860c962a34e21ae0588`
- B c1: `0xbc91ba64bcfe5eb6b465d20acfdc898d855e3ffd3db857fe537c9e4647f059ec`
- B c2: `0x0cd4df403fc2b31802eec1e17ef0bf71aeac1f3230cefb5fd4bdf352e7f7267b`
- C c1: `0xc866e58fa6fa80b03793e60ce2c7e273c769db817649e134a747e6e8d71c1468`
- Exploratory asks made while learning the tool shapes (not part of tasks A to C, $0.03): gas `0x28d89abe0ac2b03991e8e70e08616efe64b282bc5914a1f6035d557e0192914c` (output not saved), NHL `0xd4dd788b498d39d9a68c8e635729466f008a2c24688a0f529258583a1ab238f4`, future game `0x77aab5a6a2581c09de2e73ef881707af28e019562734b9c06baa7024400c4243` (`transcripts/probe-*.log`).

What the runs showed about the procedure:

- Entity and date checks matter. A miner confidently answered a future NHL game with a different sport, teams and result, every field marked `ok`. Only the entity check and the date check catch that.
- `warnings` matter. The same "conflicting information" warning appeared on the invented answer and on the first gas answer. The skill treats it as not verified until a second miner is clean.
- A receipt proves the miner answered and was paid, not that the answer is right (the invented answer has a valid receipt).
- Unverified: one receipt was fetched from `/engine/v1/signal/<hash>` and the response was empty (probably not yet settled or a different path), so receipts are not independently confirmed. Settlement is per epoch, so billing was confirmed only through `telegraph_account` `spent_today_usd`.

Files: `transcripts/taskX-asks.log` (tool calls and replies), `taskX-verdicts.log` (judge and report output), `ledger-X.json`.
