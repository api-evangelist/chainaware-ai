---
name: chainaware-ai-assess-borrower
description: Decide collateral terms for a DeFi borrower — ChainAware fraud check, then the 1-9 on-chain credit score, mapped to the provider's own collateral bands.
api: openapi/chainaware-ai-enterprise-api-openapi.yml
operations: [checkWalletFraud, getCreditScore]
method: generated
generated: '2026-09-19'
source: https://chainaware.ai/learn/api/credit-scoring-api.html
---

# Assess a borrower (ChainAware Enterprise API)

For lending protocols pricing a loan or setting a collateral ratio for a wallet, with no KYC data.

## Before you call

- `POST https://enterprise.api.chainaware.ai/...`, `x-api-key` header, `Content-Type: application/json`.
- `getCreditScore` needs the **Enterprise** subscription and supports **`ETH` only**. `checkWalletFraud` runs on `ETH`, `BNB`, `POLYGON`, `TON`, `BASE`, `TRON`, `HAQQ`.
- Reads only; no idempotency key, nothing to reverse.

## Steps

1. **`checkWalletFraud`** — `POST /fraud/check` `{"network": "ETH", "walletAddress": "<borrower>"}`. The provider's docs say to do this first: "ensure a creditworthy wallet is not also a fraud risk". Stop on `status: Fraud` or `sanctionData[].isSanctioned: true`.
2. **`getCreditScore`** — `POST /users/credit-score` `{"network": "ETH", "walletAddress": "<borrower>"}`. Read `creditData.riskRating` (integer 1-9, 9 = most trustworthy).
3. Map the score with the provider's published bands and your own risk tolerance:
   - `8-9` high trust → eligible for undercollateralised / low-collateral loans
   - `6-7` above average → reduced collateral
   - `4-5` average → standard collateral, monitor repayment
   - `2-3` below average → full overcollateralisation
   - `1` low trust → decline or maximum collateral
4. Optionally call `auditWalletBehaviour` (`POST /fraud/audit`) for `userDetails.total_balance_usd`, `wallet_age_days` and the `Prob_Borrow` / `Prob_Lend` intent signals before finalising terms.

## Obligations the provider puts on you

The Terms (https://chainaware.ai/terms/) make the lending customer responsible for consumer-credit, fair-lending, anti-discrimination and data-protection compliance in its own jurisdiction, including any adverse-action disclosure to its users, and state the score is a statistical prediction that can be wrong in an individual case. Log the score and the decision.

## Errors

`400` · `401` (`403` at the edge with no key) · `404` wallet not found · `429` (no published limits) · `500`. See `errors/chainaware-ai-problem-types.yml`.
