---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     07. Apr, 2025
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

## Price Feed: solvBTC Yield Token Rate Usd Aggregator

### Scope

Repository: https://github.com/enzymefinance/protocol

``git show bfe99ef36ac14fc5e6fd3afee3e6f664ac672522`` excluding tests.

```
contracts/release/infrastructure/price-feeds/primitives/SolvBtcYieldTokenRateUsdAggregator.sol
contracts/release/infrastructure/price-feeds/primitives/factories/SolvBtcYieldTokenRateUsdAggregatorFactory.sol
```

### Overview

This commit introduces a rate aggregator for SolvBTC yield tokens that is based on ``RateAggregatorBase``. 
Note that the `baseRate()` is defined by `getValueByShares()` (see [SolvBTCYieldToken.sol](https://github.com/solv-finance/SolvBTC/blob/56111dc7d1151f5533090a9d4f4a8287fdc6d8b2/contracts/SolvBTCYieldToken.sol#L29-L32)) which uses the result of `getNav()` (see [SolvBTCYieldTokenOracleForSFT.sol](https://github.com/solv-finance/SolvBTC/blob/43f7732c66525a4778dc7d9ac77a36263322a184/contracts/oracle/SolvBTCYieldTokenOracleForSFT.sol#L51-L59)) function.

### Risk

- Loss of funds due to incorrect valuation

### Properties Checked

- Correct usage of libraries
- Meaningful value

### Findings

- *Note*: The following assumptions are made which could not be confirmed due to a lack of documentation of the external system
    - We assume that `shares` correspond to yield token balances.
    - We assume that `getNav()` returns a value in the unit of `[solvBTC / share]` so that the computation is meaningful in the used function.
    - We assume that `getNav()` and thus `currentNav` will be in `navDecimals()` (see [SolvBTCYieldTokenOracleForSFT.sol](https://github.com/solv-finance/SolvBTC/blob/9f7133d9cc07efc09b4384fd3f653998ada87b99/contracts/oracle/SolvBTCYieldTokenOracleForSFT.sol#L62-L66)).
    - Given that `navDecimals ()` returns `IERC20(slotBaseInfo.currency).decimals()`, we assume that the currency is solvBTC.
- *Note*: We assume that the function used is not manipulateable.
- *Informational*: `getNav()` could be used directly instead of using `getValueByShares()` since the parameter would be cancelled out by the division within the function.

# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.