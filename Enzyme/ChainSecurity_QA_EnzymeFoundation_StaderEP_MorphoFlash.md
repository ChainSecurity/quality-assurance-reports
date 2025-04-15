---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     28. Nov, 2024
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

## External Position: Stader ETHx Withdrawal

### Scope

Repository: https://github.com/enzymefinance/protocol

``git diff 041d95bcf53b46cdc156ed8d885651ee091ef28c..b85907086599c87a75344556165706fcac8a1eed`` excluding interfaces and tests.

```
contracts/release/extensions/external-position-manager/external-positions/stader-withdrawals/
    StaderWithdrawalsPositionLib.sol
    StaderWithdrawalsPositionParser.sol
```

### Overview

This commit introduces an external position for the withdrawals ETHx tokens. Namely, the external position implements two actions:

- request withdrawal: Registers the intention to withdraw ETHx tokens.
- claim: Eventually, a request ID can be claimed and native ETH will be received.

### Risk

- Funds could be lost in case of bad integration.
- The valuation could be wrong, leading to bad share prices.
- Third-party risks such as share price manipulation (e.g. read-only reentrancy).

### Properties Checked

- Initialization.
- Fund flows.
- Expected functionality used for Stader.
- Expected actions available.
- Valuation returns reasonable results.

### Findings

**Notes:**

- Stader can change the address of the ETHx token which might lead to potential issues when withdrawing.
- Stader has a range for valid withdrawal amounts. Too high or too low ETHx withdrawal values could lead to reverts. It is worth noting that, if the vault has low amounts of ETHx, no withdrawal will be possible.

**Issues:**

- *Low:* DoS of Position: The external position can be temporarily DoSed. Namely, arbitrary users can push withdrawal requests for the position. Hence, maximum amount of withdrawal requests could be reached so that an actual request for ETHx held by the vault could revert.
- *Low*: The arbitrary pushing to the positions request IDs could make valuation very expensive. While the limits might make sense for ETHx, the limits might not make sense for Enzyme. As a consequence estimation of funds might be too expensive for Enzyme to handle. Note that this potential issue has not been investigated further.
- *Medium/High:* The valuation of the withdrawals not finalized could be incorrect. Namely, for not finalized amounts, the ETHx amount is returned, implying that the vault will valuate according to the price provider's price. However, Stader will eventually send the `ethFinalized` amount on claim. That amount is computed during finalization as `Math.min(requiredEth, (lockedEthX * exchangeRate) / DECIMALS)`. Hence, if the `exchangeRate` grew rapidly for some reason, the position will overestimate the position since the `requiredEth` will be the ETH amount when `requestWithdraw` was called. Ultimately, that could allow for profitable share price arbitrage.




## Asset Manager Smart Account: Morpho Flashloans

### Scope

Repository: https://github.com/enzymefinance/protocol

``git diff b85907086599c87a75344556165706fcac8a1eed..e1a53a190189fa5a9483f5be873a2d2bbf6cf67a`` excluding interfaces and tests.

```
contracts/persistent/smart-accounts/morpho-blue-flash-loan-asset-manager/MorphoBlueFlashLoanAssetManagerLib.sol
```

### Overview

This commit introduces a smart account similar to the `AaveV3FlashLoanAssetManagerLib` that will act as an asset manager. The main difference is that Morpho is used for flashloan instead of Aave.

### Risk

Same risks and trust assumptions as with the `AaveV3FlashLoanAssetManagerLib`.
Additionally, if Morpho were to be malicious, arbitrary calls could be performed.

### Properties Checked

- Validated that core logic is equal (for interactions: similar) to the Aave Flashloan Manager (access control and core logic).
- Validated access control:
  - Validated that Morpho Blue will only perform the callback on ``msg.sender``.
  - Validated that only the owner of the contract can enter the flashloan function.
- Validated that Morpho will pull funds and that the appropriate approval was given. Similarly, validated that no fees are taken on flashloans.
- Validated that the lack of a `REPAYMENT_BALANCE_BUFFER` is fine on a high level. Namely, that is due to Morpho not supporting rebasing tokens or similar.

### Findings

**Notes:**

- The Morpho Flashloan supports only one asset per flashloan call which is different from the Aave flashloan. However, in theory it could be possible to reenter the manager to retrieve multiple assets from Morpho.

**Issues:**

None found.

# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.