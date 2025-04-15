---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     03 March, 2025
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

## Enzyme V4 Shares Adapter

### Scope

Repository: https://github.com/enzymefinance/protocol

- ``git show 2da2ecbc43d89f325adfe930feebfe990122e10f`` and ``git show 428be9c27bca045c5b21ad0dc9d9420dfebbb2db`` 
- ``git show 4cc3236981546048215552ea420b0b2b0b9b69ab`` 
- Fix: ``git show 07335dea202218947b06108e5d9c44f21de3f769``

excluding interfaces and tests:

```
contracts/release/extensions/integration-manager/integrations/adapters/EnzymeV4VaultAdapter.sol
contracts/release/infrastructure/price-feeds/derivatives/feeds/EnzymeVaultPriceFeed.sol
```

### Overview

An adapter allowing Enzyme vaults to integrate with other Enzyme vaults is introduced.
Actions introduced are `BuyShares` and `RedeemSharesForSpecificAssets` allowing to call the respective functions on other vaults.

Additionally, a derivative price feed for pricing vault shares introduced.

### Risk

- Incorrect Integration: The expected function are called and the expected values are returned. 
- Denial of Service: Holding unsuitable shares might DoS some functionality.
- Leftover funds in adapter.
- Incorrect pricing. Incorrect pricing may allow for arbitrage.
- Violation of the trust model. Dependencies of funds may have implications on the funds' trust models.

### Properties Checked

- Integration with vaults is as expected.
- Proper validation of suitable vaults (validated to be Enzyme vaults).
- Read-only reentrancy considered (`callOnExtension` is called in price-feed).

### Findings

1. **Corrected** - *Informational*: On `BuyShares`, the denomination asset is passed as an argument. However, it could be retrieved from the vault proxy's comptroller. As a consequence, an unexpected asset could be passed as an argument. With prior donations to the adapter, the adapter could allow asset managers to swap assets with the managed vault.
2. *Note*: Policies should be in place that disallow integrating with unsupported vaults (e.g. policies requiring pricing shares or malicious vaults).
3. *Note*: Some funds might not be compatible. Below is a list of examples:
    - No funds should be set up with circular dependencies as this might unnecessarily DoS certain functionality.
    - A fund A holding shares of another fund B inherits the trust model of B to some degree. For example, the security guarantees provided by A's policies could be weakened by holding shares of B (e.g. B has no cumulative slippage policy while A does). In contrast, the policies for some fund C holding B's shares could remain the same (both don't have a cumulative slippage policy).
4. *Informational*: The price feed validates that the fund deployer is non-zero. However, the adapter validates that it corresponds to a given address. Ultimately, an inconsistency between the two checks exists.
5. *Note*: The dummy asset is not supported as part of `RedeemSharesForSpecificAssets`.
6. *Note*: Share prices might be manipulated. Below is a list of examples:
    - The share price might be manipulated by untrusted parties (e.g. flashloan asset manager is used with a `TransferAssetsAdapter`).
    - A malicious owner could adjust his vault to allow draining the vault that holds its vaul shares.

# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.