---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     29. September, 2025
Client:   Enzyme Foundation
---

# Extensive QA


This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.


## Topic

### Scope

Repository: https://github.com/enzymefinance/protocol

- `git show 44edc711fdc91556476d5e1eb750cc642d03eedc` excluding tests.
- Fix: `git show cd3df1161b4720e24c5d3a1220ad40bfc84fccb5`


```
contracts/release/extensions/integration-manager/integrations/adapters/BebopBlendAdapter.sol
```

### Overview

This commit introduces an integration adapter for [Bebop's PMM RFQ](https://docs.bebop.xyz/bebop/smart-contracts/pmm-rfq-smart-contract).
The adapter has:

- One supported selector for `action` (`IntegrationSelectors.ACTION_SELECTOR`).
- One supported action within `IBebopBlendAdapter.Action.SwapSingle` that allows interacting with the PMM RFQ contract's `BebopSettlement.swapSingle` function.

That allows vaults to swap a token for another one on Bebop. Note that only swaps with whitelisted makers are allowed (if list ID is non-zero).

### Risk

- Loss of provided funds through bad trades, stale orders, bad integration or bugs in Bebop. However, this is limited by policies.
- Execution of offers by other parties.
- Griefing of managers.
- Reentrancy.

### Properties Checked

- *Trades can only be executed by the target vault.* Note that only the expected taker (i.e. the adapter) will be able to execute the given trade. Since the adapter has no signing capabilities, third parties unrelated to the Enzyme ecosystem will not be able to execute. Additionally, the adapter enforces that the execution of a trade is made by the corresponding receiving vault proxy. As a consequence, a vault will only be executed by the target vault.
- *Griefing of managers.* 
- *Unsuitable order configurations.* Some provided parameters (e.g. flags and packed commands) might not be meaningful. No weird unexpected execution is seemingly possible to occur through the Bebop. Note that receiving native tokens from the maker could technically also work given received native tokens would be handled by the vault proxy.
- *Left-over tokens.* Left-over tokens should generally not occur given the Bebop code.

### Findings

**Issues**

1. **Corrected** - *Low severity:* Fees are not considered when computing the minimum output amounts. Note that the issue has been resolved by allowing the managers to specify a custom minimum incoming asset amount. Note that this does not violate the trust model as bad orders could be filled; however, that is typically regulated by vault policies.
2. **Corrected** - *Informational:* The execution does not revert if an unsupported action ID is provided. That contradicts other adapters (e.g. Paraswap). Note that the issue has been resolved by reverting for invalid action IDs.

**Notes**

1. Execution could, in theory, be griefed by malicious makers.
2. While it was checked that no left-over tokens should exist, it could make sense to try to sweep tokens from the adapter to ensure that there is no unexpected execution path that does not use all tokens in the adapter or does not send all tokens to the vault proxy.
3. Partial filling of maker offers is not possible.
4. Read-only reentrancy might be possible that could allow manipulating pricing of Enzyme vault shares.
5. Weird executions might be possible. For example, if native transfer are part of the order, donations to Bebop could in theory lead to successful execution.
6. Only fully filling orders is possible with the adapter.

# Limitations and Use of This Report

This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.