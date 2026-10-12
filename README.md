# clone-lin-gmx-fee

Experimental i64-scale LIN clone of [gmx-io/gmx-contracts](https://github.com/gmx-io/gmx-contracts) VaultUtils **getFeeBasisPoints** (USDG tax/rebate vs target weight).

- **LIN clone:** [`src/lin_gmx_fee.lin`](src/lin_gmx_fee.lin)
- **C11 oracle:** [`test/oracles/gmx_fee_c11.c`](test/oracles/gmx_fee_c11.c)
- **Proofs (results only) live in lin-open:** https://github.com/kbelludoo/lin-open
- **Event receipt:** `docs/events/EVENT_GMX_FEE_CLONE_LIN.rulel` in lin-open

This repository is the standalone LIN copy. It is **not** a GMX Vault, **not** GLP, and does **not** claim Solidity uint256 / mainnet parity. The kernel is the GMX V1 **dynamic USDG fee curve**: rebate when a mint/swap improves a token's weight versus target, tax when it worsens it. It is **not** Uniswap V2 `xy`, **not** Solidly cubic `x3y+y3x`, **not** Saddle/Curve `getD`, and **not** Maker `rpow`.

Upstream is MIT. This LIN kernel is an original uint64-scale implementation of the published `getFeeBasisPoints` formula, not a verbatim Solidity dump. `getFeeBasisPoints` / `hasDynamicFees` / `usdgAmounts` are rejected by the Solidity transpiler (`REJ_SOLIDITY_GMX_FEE`).

Pinned upstream: commit `fe55c4ba8911ae2378b50bfb9bece5ee50cf2af8`, `contracts/core/VaultUtils.sol` sha256 `ad831f3c8d675c42665c854368bc06696e1cba23bd3294f91aa2ce4c0a8fc495` blob `8e29c3ff19258d191f6d2459c110a917ba1f9906`.

Canonical goldens (uint64-scale, 128-bit product):

- at-target 10k, +2k increment, fee=100 tax=50: **105**
- underweight 5k-on-10k +2k increment: **75** rebate (flat would be 100)
- overweight 15k-on-10k +2k decrement: **75** rebate
- overweight 15k-on-10k +2k increment: **130** tax vs **100** flat (30 bps extra)
- published GMX test at-target 29700, 1000-delta: **100**; 5000-delta: **104**
- underweight 23760-on-29700 +1000: **90**
- naive i64 `(5e9*5e9)/1e6` wraps to **6553255926290**; 128-bit muldiv = **25000000000000**
