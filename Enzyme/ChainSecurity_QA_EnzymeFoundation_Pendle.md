---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     10. Jan, 2025
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

##  Pendle Adapter: refactor action namespaces

### Scope

Repository: https://github.com/enzymefinance/protocol

Commit ``7b7a6e19bfed3b6adf3d5b314c63297cdba5fea1`` excluding interfaces and tests.

```
contracts/release/extensions/integration-manager/integrations/adapters/interfaces/IPendleV2Adapter.sol
contracts/release/extensions/integration-manager/integrations/adapters/PendleV2Adapter.sol
```

### Overview

This commit updates ``enum Action`` fieds ``AddLiquidity`` to ``AddLiquidityFromUnderlying`` and ``RemoveLiquidity`` to ``RemoveLiquidityToUnderlying``. Consquently the structs for the respective actions arguments have been renamed: ``
AddLiquidityActionArgs`` -> ``AddLiquidityFromUnderlyingActionArgs`` and ``RemoveLiquidityActionArgs`` -> ``RemoveLiquidityToUnderlyingActionArgs``. 

### Risk

No risk detected.

### Properties Checked

Code change to be as described. The new code compiles successfully, indicating all required changes have been made.

### Findings

None found.

## Pendle Adapter: new Action: RemoveLiquidityToPtAndUnderlying

### Scope

Repository: https://github.com/enzymefinance/protocol

Commit ``b776668070ed04f6ed42c31df33c12474e53f7fd`` excluding interfaces and tests.

```
contracts/release/extensions/integration-manager/integrations/adapters/PendleV2Adapter.sol
```

### Overview

This commit adds an action to redeem LP shares pro-rata for PT + underlying to the Pendle Adapter. This is necessary to withdraw from an imbalanced marked where one cannot withdraw in underlying only.

### Risk

Stateless Adapter shared by all participants to interact with a third party system (Pendle). New action ``__removeLiquidityToPtAndUnderlying`` has similar/same risks as the existing ``__removeLiquidityToUnderlying`` action.

For removing liquidity this new action calls ``PENDLE_ROUTER.removeLiquidityDualSyAndPt()`` instead of ``PENDLE_ROUTER.removeLiquiditySingleSy()``. Same risks/assumption on the third party system. Trusted return values are used for further processing of the funds (syTokenAmount, ptTokenAmount).
The parameters ``_minSyOut`` and ``_minPtOut`` for the call are hardcoded to 1, effectively disabling the slippage protection. The action args allow to specify ``minWithdrawalTokenAmount`` and ``minPtAmount`` which are enforced by the IntegrationManager.

The SY tokens received are redeemed for underlying exactly as in the already existing action.
PT tokens received are transfered to the vault.

### Properties Checked

- Validated that core logic is equal (redemption of SY tokens).
- Validated that PT tokens are forwarded to the vault.
- Validated that incoming assets are properly reported.

### Findings

None found.

## Pendle External Position: new Action: MigrateToVault

### Scope

Repository: https://github.com/enzymefinance/protocol

Commit ``2214c4207f6000c889ead3e9cb7bbb5f60a35aea`` excluding interfaces and tests.

```
contracts/release/extensions/external-position-manager/external-positions/pendle-v2/PendleV2PositionLib.sol
PendleV2PositionLibBase1.sol
PendleV2PositionParser.sol
```

### Overview

This commit adds an action to move all PTs and LPTs held by the EP to the fund's vault.

### Risk

Lib:
Leaving behind untracked assets at the EP.

Parser:
Incorrect reporting of incomming assets to the vault.

Resue of existing functionality ``_pushFullAssetBalances()`` reduces the risk of bugs. To identify the tokens to be move to the vault, existing view functions are use. Existing ``getManagedAsset()`` uses the same source to identify assets held by the EP, this reduces the risk of missing assets that contributed to the EP's value.

Claiming rewards is unaffected by this new action. Rewards might be claimable even after all lp/pt positions have been transferred to the vault.


### Properties Checked

- Validated all assets held by the EP are forwarded to the vault.
- Validated that all assets contributing to the EP's valuation are forwarded to the vault.
- Effect on reward tokens.

### Findings

None found.

# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.