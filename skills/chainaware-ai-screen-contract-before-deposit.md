---
name: chainaware-ai-screen-contract-before-deposit
description: Check a token contract or liquidity pool for rug-pull risk with ChainAware before depositing, listing or routing to it.
api: openapi/chainaware-ai-enterprise-api-openapi.yml
operations: [checkRugPull, checkWalletFraud]
method: generated
generated: '2026-09-19'
source: https://chainaware.ai/learn/api/fraud-detection-api.html
---

# Screen a contract or pool before deposit (ChainAware Enterprise API)

For DEXes auto-scanning new pools, launchpads vetting listings, wallets warning users, or agents about to ape in.

## Before you call

- `POST https://enterprise.api.chainaware.ai/rug/pull-check`, `x-api-key` header, JSON body. Business or Enterprise subscription.
- The body field is named `walletAddress` in the contract even though you pass a **contract or pair address**; `network` is the uppercase enum (`ETH`, `BNB`, `POLYGON`, `TON`, `BASE`, `TRON`, `HAQQ`).
- Read only; safe to repeat.

## Steps

1. **`checkRugPull`** — `POST /rug/pull-check` `{"network": "BNB", "walletAddress": "<contract or pool>"}`.
   - Read `risk_status` (e.g. `Low Risk`) and `risk_score`, then the `risk_indicators` object — static contract flags such as `is_honeypot`, `honeypot_with_same_creator`, `can_take_back_ownership`, `is_mintable`, `hidden_owner`, `buy_tax`/`sell_tax` and friends — plus `pairAddress` (the DEX pair analysed) and `contractCreatorAddress` (nullable).
   - `404 Contract not found` means no profile exists for that address on that network.
2. **`checkWalletFraud`** on `contractCreatorAddress` when it is non-null — the deployer's own fraud probability and forensic flags are the behavioural half of the rug-pull signal.
3. Act: block or warn on honeypot/ownership flags regardless of score; otherwise apply your own threshold on `risk_score`.

## Notes

- The API docs quote 68% accuracy on new pools; the website's Rug Pull Detector V3 claims 90.1%. The API page has not been updated to say which model backs `/rug/pull-check`.
- For the deeper 11-module Token Audit (verdicts CLEAN / SUSPICIOUS / HIGH RISK / HONEYPOT / THEFT / UNVERIFIABLE, 0-100 risk score) there is no REST operation — use the MCP tools `run_token_audit` → `get_token_audit_result`, which need no API key and take **lowercase** network ids (`eth`, `bsc`, `base`, `arbitrum`, `avalanche`, `optimism`, `polygon`).

## Errors

`400` · `401` (`403` at the edge) · `404` contract not found · `429` · `500`. See `errors/chainaware-ai-problem-types.yml`.
