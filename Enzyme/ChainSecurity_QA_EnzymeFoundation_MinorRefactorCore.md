---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     04. Sept, 2025
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

## Refactoring of Core Contracts

### Scope

Repository: https://github.com/enzymefinance/protocol

`git diff v4..dev` is the expected diff to review.
At the time of writing this corresponds to `12ee243beedcf9132d328575ea3ee27c4df64856..6b7b919ac1d15e187907ddb0f286498b754bd036`.

Note that a new file has been introduced as part of the diff:

```
contracts/persistent/vault/utils/IProxiableVaultLib.sol
```

The following files were part of the scope for diffing:

```
contracts/persistent/vault/VaultLibBase1.sol
contracts/persistent/vault/VaultLibBase2.sol
contracts/persistent/vault/VaultLibBaseCore.sol
contracts/persistent/vault/utils/ProxiableVaultLib.sol
contracts/persistent/vault/utils/SharesTokenBase.sol
contracts/persistent/vault/utils/VaultLibSafeMath.sol
contracts/release/core/fund-deployer/FundDeployer.sol
contracts/release/core/fund-deployer/IFundDeployer.sol
contracts/release/core/fund/comptroller/ComptrollerLib.sol
contracts/release/core/fund/comptroller/ComptrollerProxy.sol
contracts/release/core/fund/comptroller/IComptroller.sol
contracts/release/core/fund/vault/VaultLib.sol
contracts/release/extensions/IExtension.sol
contracts/release/extensions/external-position-manager/ExternalPositionManager.sol
contracts/release/extensions/fee-manager/FeeManager.sol
contracts/release/extensions/fee-manager/IFee.sol
contracts/release/extensions/fee-manager/IFeeManager.sol
contracts/release/extensions/integration-manager/IIntegrationManager.sol
contracts/release/extensions/integration-manager/IntegrationManager.sol
contracts/release/extensions/policy-manager/PolicyManager.sol
contracts/release/extensions/utils/ExtensionBase.sol
contracts/release/extensions/utils/PermissionedVaultActionMixin.sol
contracts/release/infrastructure/FundValueCalculator.sol
contracts/release/infrastructure/gas-relayer/GasRelayPaymasterFactory.sol
contracts/release/infrastructure/gas-relayer/GasRelayPaymasterLib.sol
contracts/release/infrastructure/gas-relayer/GasRelayRecipientMixin.sol
contracts/release/infrastructure/gas-relayer/IGasRelayPaymaster.sol
contracts/release/infrastructure/gas-relayer/bases/GasRelayPaymasterLibBase1.sol
contracts/release/infrastructure/gas-relayer/bases/GasRelayPaymasterLibBase2.sol
contracts/release/infrastructure/price-feeds/derivatives/AggregatedDerivativePriceFeedMixin.sol
contracts/release/infrastructure/price-feeds/primitives/ChainlinkPriceFeedMixin.sol
contracts/release/infrastructure/protocol-fees/ProtocolFeeTracker.sol
contracts/release/infrastructure/value-interpreter/ValueInterpreter.sol
contracts/release/peripheral/DepositWrapper.sol
```

### Overview

The following is introduced:

- Refactoring of authors and e-mail addresses:
    - Author: `@author Enzyme Council` to `@author Enzyme Foundation`.
    - Copyright: `(c) Enzyme Council <council@` to `(c) Enzyme Foundation <security@`.
- Moving file `contracts/release/off-chain/FundValueCalculator.sol` to `contracts/release/infrastructure/FundValueCalculator.sol`.
- `ProxiableVaultLib.sol` now implements a new interface `IProxiableVaultLib.sol`.

### Risk

- Breaking compilation due to incorrect interface.
- Additional unexpected changes.
- Unexpected modifications.

### Properties Checked

- No additional changes.
- Compilation works.
- All modification as described in [Overview](#overview).

### Findings

None found. The approach was automated with the script provided in the [Attachements](#attachements)

# Limitations and Use of This Report

This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.

# Attachements

This section provides the script that was used for validating the changes.
Please validate the script for safety before running it.

```bash
# checkout the HEAD of dev at the time of writing
git checkout 6b7b919ac1d15e187907ddb0f286498b754bd036 || exit 1

# undo the refactoring of authors and e-mail addresses with sed (back to the supposed version as in v4)
find contracts -type f -name "*.sol" -exec sed -i \
  -e 's/author Enzyme Foundation/author Enzyme Council/g' \
  -e 's/(c) Enzyme Foundation <security@/(c) Enzyme Council <council@/g' {} +

# move the FundValueCalculator to the place where it was in v4 to allow easier diffing
mv contracts/release/infrastructure/FundValueCalculator.sol contracts/release/off-chain/FundValueCalculator.sol

# git diff all relevant files and inspect output manually. (only changes due to IProxiableVaultLib will be visible)
git diff v4 -- \
    contracts/persistent/vault/VaultLibBase1.sol \
    contracts/persistent/vault/VaultLibBase2.sol \
    contracts/persistent/vault/VaultLibBaseCore.sol \
    contracts/persistent/vault/utils/ProxiableVaultLib.sol \
    contracts/persistent/vault/utils/SharesTokenBase.sol \
    contracts/persistent/vault/utils/VaultLibSafeMath.sol \
    contracts/release/core/fund-deployer/FundDeployer.sol \
    contracts/release/core/fund-deployer/IFundDeployer.sol \
    contracts/release/core/fund/comptroller/ComptrollerLib.sol \
    contracts/release/core/fund/comptroller/ComptrollerProxy.sol \
    contracts/release/core/fund/comptroller/IComptroller.sol \
    contracts/release/core/fund/vault/VaultLib.sol \
    contracts/release/extensions/IExtension.sol \
    contracts/release/extensions/external-position-manager/ExternalPositionManager.sol \
    contracts/release/extensions/fee-manager/FeeManager.sol \
    contracts/release/extensions/fee-manager/IFee.sol \
    contracts/release/extensions/fee-manager/IFeeManager.sol \
    contracts/release/extensions/integration-manager/IIntegrationManager.sol \
    contracts/release/extensions/integration-manager/IntegrationManager.sol \
    contracts/release/extensions/policy-manager/PolicyManager.sol \
    contracts/release/extensions/utils/ExtensionBase.sol \
    contracts/release/extensions/utils/PermissionedVaultActionMixin.sol \
    contracts/release/infrastructure/FundValueCalculator.sol \
    contracts/release/infrastructure/gas-relayer/GasRelayPaymasterFactory.sol \
    contracts/release/infrastructure/gas-relayer/GasRelayPaymasterLib.sol \
    contracts/release/infrastructure/gas-relayer/GasRelayRecipientMixin.sol \
    contracts/release/infrastructure/gas-relayer/IGasRelayPaymaster.sol \
    contracts/release/infrastructure/gas-relayer/bases/GasRelayPaymasterLibBase1.sol \
    contracts/release/infrastructure/gas-relayer/bases/GasRelayPaymasterLibBase2.sol \
    contracts/release/infrastructure/price-feeds/derivatives/AggregatedDerivativePriceFeedMixin.sol \
    contracts/release/infrastructure/price-feeds/primitives/ChainlinkPriceFeedMixin.sol \
    contracts/release/infrastructure/protocol-fees/ProtocolFeeTracker.sol \
    contracts/release/infrastructure/value-interpreter/ValueInterpreter.sol \
    contracts/release/peripheral/DepositWrapper.sol \

# remove the moved files and reset
rm contracts/release/off-chain/FundValueCalculator.sol
git reset --hard
```