---
name: Discover RockSolid vaults and their yield
description: Browse RockSolid liquid vaults, inspect a vault's strategy allocations for a period, and read its calculated APR and TVL.
api: openapi/rocksolid-vaults-openapi.yml
operations: [listVaults, getVault, listAllocationPeriods, getVaultAllocations, getVaultApr]
---

# Discover RockSolid vaults and their yield

Use the public, read-only RockSolid Vaults API (`https://app-integration.rocksolid.network/api`) to explore liquid vaults. No authentication is required.

## Steps

1. **List vaults** — call `listVaults` (`GET /vaults`). Keep only entries where `is_active` is `true`. Each item carries `vault_address`, `symbol`, `underlying_asset`, `chain`, fees (`aum_fee`, `performance_fee` in basis points), and `latest_performance`.
2. **Inspect a vault** — call `getVault` (`GET /vaults/{vault_address}`) with a `vault_address` from step 1 to get full metadata, curators, rewards, and up to 30 performance-history records.
3. **Pick a period** — call `listAllocationPeriods` (`GET /vaults/allocation-periods`) and select an `allocation_period_id` (results are newest-first).
4. **Read allocations** — call `getVaultAllocations` (`GET /vaults/{vault_address}/allocations?period-id={id}`). `allocation` and `strategyApr` are decimals (0.20 = 20%). An optional `performance_note` may accompany the snapshots.
5. **Read yield** — call `getVaultApr` (`GET /vaults/{vault_address}/apr`), optionally with `days_to_compute` (default 14). `tvlRaw` is wei (string); `apr` is a percentage string. For rETH vaults, also read `baseAprAgainstEth` / `strategyAprAgainstEth` / `totalAprAgainstEth`.

## Rules

- **Read-only**: every operation is a `GET`; there are no write/idempotency concerns. Deposits and withdrawals happen on-chain against the ERC-7540 vault contracts, not through this API.
- **Numbers**: treat wei/`*_raw` values as strings (do not parse as native floats); convert basis-point fees by dividing by 100.
- **Required param**: `getVaultAllocations` requires `period-id` — a missing/invalid value returns `400`.
- **Errors**: responses use a custom `{error, message, timestamp, details}` envelope (not RFC 9457). Handle `400` (bad param), `404` (unknown vault/period), and `500` (server/subgraph failure).
- **Caching**: most endpoints cache for ~5 minutes; do not expect sub-minute freshness.
