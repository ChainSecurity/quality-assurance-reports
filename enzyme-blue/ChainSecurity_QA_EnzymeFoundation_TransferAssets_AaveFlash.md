---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     October 2024
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

# Adapter: Transfer Assets

## Scope

Repository: https://github.com/enzymefinance/protocol

``git show e036b304485f4bc6ca4a0ed1eb4e4b8481da951b`` excluding interfaces and tests.

## Overview

The adapter allows the fund manager to send assets of the fund to an arbitrary address. The state-changing methods of the adapter can only be invoked by the integrations
manager. The adapter can only get access to the vault’s funds if it holds an approval from the vault. Note that the approval is fully withdrawn at the end of a transaction.

## Risk
From the overview above, we conclude that the risk to the system is **limited** given a non-malicious manager. As managers always act in the interest of their users, any potential problem could also be resolvable with the collaboration of the managers. We also assume that appropriate policies are in place to limit the actions of the manager.

## Properties Checked

- Access control
- Remaining approvals: Approvals for the adapter are set to 0.
- Correct preparation of spent assets.
- Trust: Adds extra trust to the manager who needs to have appropriate policies and the throttle

# Aave Flashloans

## Scope

Repository: https://github.com/enzymefinance/protocol

``git show c5acff2b309b8940de604d0aecdfbaa12ce332a2`` excluding interfaces and tests.

## Overview

A peripheral contract that facilitates vault actions with a flashloan. The contract has asset manager privileges.

## Risk

Such malfunctioning can expose a vault to threats as it has high privileges on the vault funds. A problematic contract could enable an arbitrary user to get access to the vault's funds. Therefore, we deem the risk to be **high**.

## Properties

- Initialization: Even initializing the owner to 0 should be fine as no flashloans can be executed. Allows for re-initialization. We assume that the fund owner will only whitelist properly set proxies.
- Accidentally setting the `borrowedAssetRecipient` to 0 is fine.
- No arbitrary user can invoke `executeOperation`.
- `initiator` is checked so no calls can be initiated from another contract.
- `REPAYMENT_BALANCE_BUFFER` makes sense. Its value is arbitrary but could only lead to DoS.
- The contract can’t be DoSed by donating assets to it. All flash loaned assets are moved to the vault and only the required part should returned.
- Any weird execution path would require a malicious user to eventually have been whitelisted by the fund manager.
- It’s ok that the call target is never sanitized.
- Policies are run for each action so the damage from batched calls is limited.


## Open Questions

- (Very minor) What happens with the remaining assets in the contract? Should they be transferred back to the vault?

# External Position update: Add claiming rewards to Aave v3 debt positions

## Scope

Repository: https://github.com/enzymefinance/protocol

``git show 461bf0e9e2413b58ad76370764531661188e6f19`` excluding interfaces and tests.


## Overview

It extends the logic of the respective peripheral contract to allow it to claim rewards.

## Risk

Given that the rest of the contract functions correctly, adding this functionality exposes the system to **low** risks. A wrong implementation could prevent the peripheral contract from being able to claim the rewards.

## Properties

- No sanitization of input assets. AaveV3 should just revert in such a case.
- `claimRewards` interface is used correctly.
- Reward token is sanitized (sometimes this is skipped in Enzyme see ConvexVoting)

# Dispatcher-owned beacon proxy factory

## Scope

Repository: https://github.com/enzymefinance/protocol

``git show 28311ab35013ccbe0cdbb19fb0d6cafbbabfa128`` excluding interfaces and tests.

## Overview

This is a peripheral contract with no special privileges. It inherits from the already audited DispatcherOwnedBeacon.

## Risk

Assuming no issues with the parent contract the risk is **very low**.

## Properties

- It’s okay that anyone can deploy a new contract.
- Access control only when setting implementation.
- Passing arbitrary construction data is not problematic. The deployed contract should not be used.

# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.