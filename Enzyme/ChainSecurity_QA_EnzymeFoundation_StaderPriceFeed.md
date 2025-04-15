---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     19. Dec, 2024
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

# Stader Staking Price Feed

## Scope

Repository: https://github.com/enzymefinance/protocol

``git show 64baa55fd6d68c72d5be0fdf7bb72cf110a4dea4`` excluding interfaces and tests.

## Overview

This commit introduces a price feed for Stader's ETHx. The price feed can be used to evaluate the GaV of a vault that holds ETHx.

## Risk

We assume Stader's oracle implementation is correct, cannot be manipulated and Stader's trusted nodes don't misbehave and submit new exchange rates frequently. Therefore, risks only involve wrong usage of the returned data within Enzyme.

## Properties Checked

* Possible manipulations of Stader price (best effort investigation) via changing token's supply or impact of black swan events.

* Possibility to check staleness of data provided by Stader (via block.number).

* Impact of exchange rate being 10**18 when Eth balance on Stader is 0. This case is extremely unlikely.

* Revert possibilities of the oracle that could lead to DoSing Enzyme Vault.


## Issues

None

## Open Question

The StaderOracle.getExchangeRate() returns the block.number in the ExchangeRate struct of the last submission. Have you considered using this to check for staleness somehow?

# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.