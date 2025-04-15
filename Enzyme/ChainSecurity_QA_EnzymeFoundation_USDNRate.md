---
Title:    Extensive QA
Author:   ChainSecurity
Date:     03. Apr, 2025
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

## Price Feed: SmarDex USDN Native Rate Usd Aggregator

### Scope

Repository: https://github.com/enzymefinance/protocol

``git show 85a5afe31737c99230ee351002577b9e6b3a9e12`` and `git show 259f3cad63922bbea3dc3cff1289e058e2a39e88` excluding interfaces and tests.

```
contracts/release/infrastructure/price-feeds/primitives/SmarDexUsdnNativeRateUsdAggregator.sol
```

### Overview

This commit introduces a rate aggregator for USDN that is based on ``RateAggregatorBase``. Note that the `baseRate()` is defined by the `usdnPrice()` (see [UsdnProtocolVault.sol](https://github.com/SmarDex-Ecosystem/usdn-contracts/blob/6060feb476a51294dd6d25d452cceca41961cc64/src/UsdnProtocol/UsdnProtocolVault.sol#L9)) which uses the result of the `parseAndValidatePrice()` (see [OracleMiddleware.sol](https://github.com/SmarDex-Ecosystem/usdn-contracts/blob/6060feb476a51294dd6d25d452cceca41961cc64/src/OracleMiddleware/OracleMiddleware.sol#L72)) function.


### Risk

- Loss of funds due to incorrect valuation
- Potential Mnaipulation of price feed

### Properties Checked

- Middleware returns 18 decimal price.
- The price retrieved from middleware works with `usdnPrice()`
- Meaningful value

### Findings

- *Note*: The below assumptions were confirmed by the USDN team.
    - `vaultBalance` in [_calcUsdnPrice](https://github.com/SmarDex-Ecosystem/usdn-contracts/blob/6060feb476a51294dd6d25d452cceca41961cc64/src/UsdnProtocol/libraries/UsdnProtocolVaultLibrary.sol#L1099) is in asset decimals.
    - The middleware will always return a price with 18 decimals. Note that for the QA we only considered the `OracleMiddleware` contract and no other contracts.
    - The state reflects the expected state.
    -  While `parseAndValidatePrice()` returns a safe price, we assume that `usdnPrice` is not manipulateable.
- *Note*: Note that the USDN team mentioned that `usdnPrice` might be manipulateable but that manipulations are not instant.
- *Note*: The price feed could be DoSed in certain scenarios. Namely, within the valuation the function `vaultAssetAvailableWithFunding` could revert due to `s._lastUpdateTimestamp > block.timestamp` which might occur under certain circumstances (Pyth returning a price for a future timestamp). However, according to the USDN team that is highly unlikely.

# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.