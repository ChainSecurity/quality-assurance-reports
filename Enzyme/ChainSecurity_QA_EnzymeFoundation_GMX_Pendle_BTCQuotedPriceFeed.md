---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     November 2024
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

# GMX refactor pt1: move some logic into a delegate-called lbirary

## Scope

Repository: https://github.com/enzymefinance/protocol

``git show e031b39f458960b4e82aaea6057caac7e5aaeb46`` excluding interfaces and tests.

## Overview

This commit refactors the ``GMXV2LeverageTradingPositionLib``. It introduces ``GMXV2LeverageTradingPositionLibManagedAssets`` which the library delegates to to query ``getManagedAssets()`` and ``GMXV2LeverageTradingPositionMixin``.

## Risk

Assuming that the original implementation of the external position has no bugs, the refactoring shouldn't increase the risk for the position.

## Properties Checked

- Constants have been moved to the correct contracts.
- The implemented logic is the same. 

## Issues

INFO: ``GMXV2LeverageTradingPositionLib::CHAINLINK_PRICE_FEED_PROVIDER`` is never used

# GMX refactor pt2: Update with GMX's new contracts

## Scope

Repository: https://github.com/enzymefinance/protocol

``git show cfdc6459105740c566b251950ed98721f1de4d35`` excluding interfaces and tests.

## Overview

It refactors GMXV2 to position to work with GMX's new contracts.

## Risk

Assuming that the previous implementation of the external position is correct and the changes on the GMX's side are not very big, the refactor shouldn't expose Enzyme to big risks.

## Properties Checked

- Impact of ``afterOrderExecution`` failure:
- Possibility of using collateral outside the asset universe
- Could the changes affect the fund valuation?
- Should ``validFromTime`` new parameter affect the fund valuation?
- Judging from GMX changelog, could a change break the logic of the external position?


# BtcToEthQuotedSimulatedAggregator

## Scope

Repository: https://github.com/enzymefinance/protocol

``git show 362b5364187b66eb93efc2e1fa4a7c25a6ca3388`` excluding interfaces and tests.

## Overview

Implements a simulated aggregator reporting the rate between two base assets of two Chainlink-like aggregators that share the same quote asset. 

## Risk

A malfunctioning oracle can expose the system to great risk as it can lead to wrong fund valuation. However, the implemented logic in this particular case is not complex therefore the risk is limited.

## Properties Checked

- Correctness of conversion logic and precision
- Possibility of underflows (see open question)
- choice of returned values

## Open Questions:

- Why is ``CONVERSION_FACTOR`` an ``int256``?
- Have you considered the edge case case where: ``PRECISION_DECIMALS + QUOTE_ASSET_AGGREGATOR.decimals() - BASE_ASSET_AGGREGATOR.decimals()`` underflows? It should be very unlikely.

# PendleV2Adapter

## Scope

Repository: https://github.com/enzymefinance/protocol

``git show 424a3ed62b588f8beaedb26ef3f86c86b0452daa`` excluding interfaces and tests.

## Overview

It implements an adapter from Pendle.fi V2. It allows funds, to buy and sell principal tokens and add or remove liquidity.

## Risk

Given the trust assumptions for the fund manager, the adapter shouldn't expose the system to more risks. There are missing checks regarding the Pendle pools a user is allowed to interact with. However, this should be fine given that Enzyme will only whitelist assets for trusted pools.

## Properties Checked

- Compared to PendleV2 external position
- Potential dust after interactions
- Access control
- Impact of missing sanity checks for tokens (see syToken validation on the external position)

# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.