---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     06. May, 2025
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

## Price Feed: Pegged Rate Deviation Aggregator

### Scope

Repository: https://github.com/enzymefinance/protocol

``git show 3438a01f711b87e9609828dff2205cbf6fa82fae`` excluding interfaces and tests.

```
contracts/release/infrastructure/price-feeds/primitives/PeggedRateDeviationAggregator.sol
contracts/release/infrastructure/price-feeds/primitives/factories/PeggedRateDeviationAggregatorFactory.sol
```

### Overview

This commit introduces a rate aggregator for pegged assets that is based on ``PeggedRateDeviationAggregator``. Note that the `idealRate()` is defined as 1 (in terms of the price feeds precision).

Additionally, `PeggedRateDeviationAggregatorFactory` allows deploying oracles for assets pegged to ETH or USD (with or without quote conversion and inversion).

### Risk

- No significant risks assuming underlying contracts are correct.
- Assumed to be used for suitable tokens.


### Properties Checked

- Correct configuration.
- Meaningful `idealRate`.

### Findings

None found. 

## Adapter: Paraswap V6

### Scope

Repository: https://github.com/enzymefinance/protocol

``git show 73d7d8bed2429f38d68f98de8334e2aa6868fc9d` excluding interfaces and tests.

A second version was then provided: ``git show 5c25b16527a2db7b23428d47c31ce1864acec070`

```
contracts/release/extensions/integration-manager/integrations/adapters/ParaSwapV6Adapter.sol
```

### Overview

This commit introduces an adapter for integrating with ParaSwapV6. The actions `SwapExactAmountIn` and `SwapExactAmountOut` are provided. Both perform the naming.

### Risk

- Left-over funds.
- Abuse by privileged parties if no policies in place.
- Incorrect Paraswap integration. However, that is limited to the funds sent to Paraswap in the worst case.

### Properties Checked

- Parameters are meaningful.
- Calls are as expected.
- Left-Over funds are moved to the vault proxy.

### Findings

- *Resolved* & *Low*: `__swapExactAmountIn` will approve the V6 AugustusSwapper only with `fromAmount`. However, the AugustusSwapper will try to pull the full balance since that is the argument passed to the swap function. In the second version the issue was resolved.
- *Note*: There is a potential mismatch between the ParaSwap slippage protection and the internal one. Note that there are corner cases where the ParaSwap slippage protection will not trigger a revert where the local one will (even though the same parameters are used).

## External Position: AaveV3 MerklDistributor Rewards

### Scope

Repository: https://github.com/enzymefinance/protocol

``git show 49f7164e74edb928c50e5938f0fffcbd9989f038`` excluding interfaces and tests.


Later the following commits were added: `git show 678b139738b3142b26f86fd7515341f6449518ae` and `git show ff74c75a90923fa94d05e8c05fa2b739d53053fa`.

```
contracts/release/extensions/external-position-manager/external-positions/aave-v3-debt/AaveV3DebtPositionDataDecoder.sol
contracts/release/extensions/external-position-manager/external-positions/aave-v3-debt/AaveV3DebtPositionLib.sol
contracts/release/extensions/external-position-manager/external-positions/aave-v3-debt/AaveV3DebtPositionParser.sol
```

### Overview

This commit introduces a new action to Aave V3 Debt Positions. More precisely, rewards from a ``MerklDistributor`` (see [merkl.xyz](https://merkl.xyz/)) can be retrieved.

### Risk

- Potential of reward claiming by external parties.

### Properties Checked

- Rewards claimed by external parties can still be claimed (i.e. sweeping).

### Findings

- *Note*: Privileged addresses might claim rewards on behalf of the external position. However, sweeping allows to claim in such cases.
- *Issue*: **Resolved**. Client reported an issue in the initial version where claiming for the same token multiple times (e.g. updated roots) could fail. That was resolved in the second commit. However, the second commit presented an edge case where the array of claims could contain the same claim multiple times. While this would have led to reverts in most cases (failing transfers similar to before), a specific case could have led to unwanted modifications of the position health (e.g. reward token is aToken). In the latest version the issue was resolved.


## Price Feed: ERC4626 Rate Aggregator

### Scope

Repository: https://github.com/enzymefinance/protocol

``git show 924952c31038af36fd35129ce93dfbece3b62303`` excluding tests and interfaces.

```
contracts/release/infrastructure/price-feeds/primitives/ERC4626RateAggregator.sol
contracts/release/infrastructure/price-feeds/primitives/factories/ERC4626RateAggregatorFactory.sol
```

### Overview

This commit introduces a rate aggregator for ERC-4626 tokens with the option of quote asset conversion and inversion. The aggregator is based on ``RateAggregatorBase`` and provides the ``baseRate()`` ``ERC4626.convertToAssets()``.

### Risk

- Incorrect usage of libraries.
- Incorrect price retrieval for ERC-4626.

### Properties Checked

- Correct usage of libraries.
- Suitable pricing for ERC-4626 in the context of Enzyme.

### Findings

- *Note*: Some ERC-4626-like tokens might be unsuitable for the price feed (e.g. delayed withdrawals).

# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.