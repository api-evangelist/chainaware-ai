---
name: chainaware-ai-screen-wallet-at-connect
description: Gate a connecting wallet with ChainAware's Enterprise API — fraud check first, then the full behavioural audit and (Enterprise tier) the segment score — using the contract's real operations, network enums and error rules.
api: openapi/chainaware-ai-enterprise-api-openapi.yml
operations: [checkWalletFraud, auditWalletBehaviour, getWalletSegment]
method: generated
generated: '2026-09-19'
source: https://chainaware.ai/learn/api/fraud-detection-api.html
---

# Screen a wallet at connect (ChainAware Enterprise API)

Use this when a dApp, lending protocol or agent needs to decide — in one round trip — whether to let a wallet in, what to show it, and how much to trust it.

## Before you call

- Base URL `https://enterprise.api.chainaware.ai`. Every operation is `POST` with `Content-Type: application/json`.
- Send your key as the `x-api-key` header. Keys come from https://chainaware.ai/profile on a Business or Enterprise subscription. Without one the edge returns `403 {"message":"Forbidden"}`; with a bad one the contract says `401`.
- Use the **uppercase** network ids the contract's enums declare (`ETH`, `BNB`, `POLYGON`, `TON`, `BASE`, `TRON`, `HAQQ` for fraud; `ETH`, `BNB`, `BASE`, `HAQQ`, `SOLANA` for the audit). Ignore the lowercase ids on the overview page — they contradict the enum.
- No idempotency key exists and none is needed: these are reads. A retry is safe (see `conventions/chainaware-ai-conventions.yml`).

## Steps

1. **`checkWalletFraud`** — `POST /fraud/check` with `{"network": "ETH", "walletAddress": "<address>"}`.
   - Read `status`: `Fraud` → block; `New Address` → insufficient history, treat cautiously; `Not Fraud` → continue.
   - `probabilityFraud` is a **string** decimal. The provider's own bands: `0.00-0.20` proceed, `0.21-0.50` caution, `0.51-0.80` manual review, `0.81-1.00` block.
   - `forensic_details` carries 19 flags (`sanctioned`, `mixer`, `phishing_activities`, …) and `sanctionData[]` any list matches — log them; they are the AML evidence.
   - `lastChecked` / `checked_times` tell you how fresh the cached score is. Pass `"calculate": true` only when you need a live recalculation.
2. **`auditWalletBehaviour`** — `POST /fraud/audit` with the same body (Business or Enterprise).
   - Returns the fraud fields again plus `experience` (0-100), `intention.Value` (14 `Prob_*` signals rated High/Medium/Low), `categories[]`, `protocols[]`, `recommendation.Value[]`, `riskProfile[]`, `userDetails{wallet_age_days,total_balance_usd,transaction_count,wallet_rank}`.
   - Route onboarding on `experience` (≤25 beginner … ≥76 expert) and surface products from the `Prob_*` signals.
3. **`getWalletSegment`** — `POST /segmentation/wallet-segment` (Enterprise only) when you need the compact segment + quality score rather than the whole profile.

## Errors

`400` bad network/address · `401` key missing or invalid (`403` from the edge when absent) · `404` wallet not found on that network · `429` rate limited (no numbers or `Retry-After` published — back off) · `500` retry shortly. Full catalog: `errors/chainaware-ai-problem-types.yml`.

## Alternatives on other surfaces

The same three checks exist as MCP tools `predictive_fraud` / `predictive_behaviour` (no segment tool) and as A2A skills `fraud_check` / `fraud_audit` / `wallet_segment` paid per call with x402 ($0.15 USDC on Base) — see `mcp/chainaware-ai-tool-crosswalk.yml`.
