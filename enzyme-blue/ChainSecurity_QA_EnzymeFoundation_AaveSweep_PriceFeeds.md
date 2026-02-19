---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     07. Feb, 2025
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

## External Position: Aave V3 Sweep Action

### Scope

Repository: https://github.com/enzymefinance/protocol

``git diff b776668070ed04f6ed42c31df33c12474e53f7fd..58993538c08c0231691ff6c7a6e03b73dce78a7c`` excluding tests.

```

contracts/release/extensions/external-position-manager/external-positions/aave-v3-debt/AaveV3DebtPositionDataDecoder.sol
contracts/release/extensions/external-position-manager/external-positions/aave-v3-debt/AaveV3DebtPositionLib.sol
contracts/release/extensions/external-position-manager/external-positions/aave-v3-debt/AaveV3DebtPositionParser.sol
contracts/release/extensions/external-position-manager/external-positions/aave-v3-debt/IAaveV3DebtPosition.sol
```

### Overview

This commit introduces a new action ``Sweep`` for the Aave V3 external position. The action allows for withdrawing arbitrary tokens, excluding aTokens, from the position.

### Risk

- Manipulation of valuation by sending out held tokens.
- Bad assets passed leading to unwanted results.

### Properties Checked

- No aToken can be sent out.
- Debt tokens cannot be sent out (as such tokens are not transferrable).
- Valuation remains unaffected.

### Findings

None found.


## Rate Aggregator Base Contracts

### Scope

Repository: https://github.com/enzymefinance/protocol

``git diff 58993538c08c0231691ff6c7a6e03b73dce78a7c..7e6892d33b089c8623febad2ddcd9500cec397e8`` excluding interfaces and tests.

```
contracts/release/infrastructure/price-feeds/primitives/utils/AggregatorRateDeviationMixin.sol
contracts/release/infrastructure/price-feeds/primitives/utils/RateAggregatorBase.sol
contracts/release/infrastructure/price-feeds/primitives/utils/RateEthAggregatorBase.sol
contracts/release/infrastructure/price-feeds/primitives/utils/RateUsdAggregatorBase.sol
contracts/release/infrastructure/price-feeds/utils/PriceFeedHelpersLib.sol
```

### Overview

This commit introduces a set of helper contracts for price feeds.

The abstract contract `RateAggregatorBase` implements the Chainlink interface required by Enzyme for a price feed with `DECIMALS` decimals.
It implements an abstract function `baseRate`, that should return the rate, its precision and its timestamp. The `latestRoundData` function then computes the price as follows:

1. Compute the base rate.
2. If a quote oracle is defined, get the rate from the oracle along with the timestamp. Its precision is computed in the constructor (`QUOTE_CONVERSION_AGGREGATOR_PRECISION`) (e.g. BTC / ETH).
3. If the price is inverted, compute the inverse price so that it has precision `INVERTED_RATE_PRECISION` (ETH / BTC computed to BTC / ETH)
4. Combine the base rate result with the quote rate (e.g. USD / BTC and BTC / ETH to receive USD / ETH) and scale to the `PRECISION` defined by `DECIMALS`.
5. The timestamp will be the oldest timestamp.

`RateEthAggregatorBase` and `RateUsdAggregatorBase` inherit from `RateAggregatorBase` and fix the decimals to 18 and 8, respectively.

`AggregatorRateDeviationMixin` is an abstract mix-in contract defining an internal function `__rateByDeviation` which takes an ideal rate, its precision and its timestamp as arguments. It then compares this rate with a market rate retrieved from `MARKET_AGGREGATOR_ADDRESS`. As long as the deviation does not exceed the maximum relative error `DEVIATION_TOLERANCE_BPS`, the ideal rate is returned. Otherwise, the market rate is returned. Note that the precision of the result is always in the precision of the ideal rate and that the timestamp is always the lowest timestamp.

The `PriceFeedHelpersLib` implements helper functions for converting data in the contracts above.


### Risk

Mispricing assets due to 

- Incorrect conversion of precision
- Incorrect conversion of price feed results (e.g. inverting price feeds incorrectly).

Temporary DoS potential due to potential overflows.

### Properties Checked

- Validated that the core logic follows the expected process.
- Validated that the local variable's for precisions match the expected value depending on the branch taken.
- Validated that the combination of two price feeds is done as expected. Checked similarity to `TwoAggregatorsWithCommonQuoteSimulatedAggregator`.
- Validated that the inversion of price feeds is reasonable. Checked similarity to `UsdEthSimulatedAggregator`. 

### Findings

None found.

## Converted quote aggregator

### Scope

Repository: https://github.com/enzymefinance/protocol

``git diff 7e6892d33b089c8623febad2ddcd9500cec397e8..92bfe182a0876c0e758bd709818c8fc793d30ea6`` and ``git show 259f3cad63922bbea3dc3cff1289e058e2a39e88..cf8ec107180f85fbad99ff550e14af58d41851fe`` excluding interfaces and tests.

```
contracts/release/infrastructure/price-feeds/primitives/ConvertedQuoteAggregator.sol
contracts/release/infrastructure/price-feeds/primitives/factories/ConvertedQuoteAggregatorFactory.sol
```

### Overview

This commit introduces an implementation `ConvertedQuoteAggregator` for the abstract contract `RateAggregatorBase`.
The `baseRate` function returns the price returned from a source oracle (Chainlink-like) along with its precision and its timestamp so that the `latestRoundData` will return the rate derived from Chainlink-like source and quote oracles (e.g. USD / BTC and BTC / ETH to receive USD / ETH)

`ConvertedQuoteAggregatorFactory` can deploy the `ConvertedQuoteAggregator` permissionlessly.

### Risk

- Unsuitable `baseRate` implementation.

### Properties Checked

- Validated that the function returns the result of the underlying oracle accordingly.

### Findings

None found.

# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.