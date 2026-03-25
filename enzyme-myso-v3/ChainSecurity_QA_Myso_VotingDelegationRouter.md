---
Title:    Extensive QA
Author:   ChainSecurity
Date:     24. Mar, 2026
Client:   Enzyme
---

# Extensive QA


This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system's funds, and highlights areas that may require deeper analysis like a full audit or other security measures.


## Voting Delegation Through Router

### Scope

Repository: https://github.com/mysofinance/v3

`git diff 6feef999..e5480e64`:

```
contracts/Escrow.sol        [MODIFY]
contracts/Router.sol        [MODIFY]
contracts/interfaces/IRouter.sol  [MODIFY]
```

### Overview

Adds `handleOnChainVoting` and `handleOffChainVoting` wrappers to the `Router`. The indirection and access control follow the same pattern as `withdraw`: the `Router` validates `isEscrow[escrow]` and that `msg.sender` is the escrow owner, then forwards to the `Escrow`. In `Escrow.sol`, the sender check on both voting functions is relaxed from `owner`-only to `router || owner`, matching how `handleWithdraw` already accepts the `Router` as a caller.

### Trust Model

The `Router` is now an additional trusted caller on the `Escrow` voting delegation functions. However, since the `Router` only forwards calls after verifying that `msg.sender` is the escrow owner, the effective trust model is unchanged: escrow owners remain fully trusted for their respective escrow and are the sole parties that can trigger voting delegation.

### Risk

* **Incomplete change or incorrect logic.** Incorrect integration between `Router` and `Escrow`, e.g., the `Router` forwarding with wrong parameters, or insufficient changes on the `Escrow` side (missing the `router` in the sender check), would cause delegation via the `Router` to revert.
  * *QA Confidence*: High. Both `handleOnChainVoting` and `handleOffChainVoting` in `Escrow.sol` received the identical `msg.sender != router && msg.sender != owner` guard. Both `Router` wrappers forward to the correct respective `Escrow` function with matching signatures.
  * *System Risk*: Low. The consequence would be a DoS on Router-based delegation; direct delegation by the owner on the `Escrow` remains unaffected, and no funds are at risk.

* **Incorrect access control.** A bug in the `Router` or `Escrow` access control could allow an unauthorized party to delegate voting rights on an escrow they do not own.
  * *QA Confidence*: High. The `Router` checks `isEscrow[escrow]` and `msg.sender == IEscrow(escrow).owner()` before forwarding, the same pattern used by `withdraw`. The `Escrow` independently verifies the caller is `router` or `owner`. Both layers must be bypassed for unauthorized delegation.
  * *System Risk*: Medium. Unauthorized voting delegation does not put escrowed funds at risk, but could allow an attacker to influence governance decisions using voting power they do not own.

### Findings

None.

---

# Limitations and Use of This Report
This report was created at the client's request and is provided "as-is." The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client's discretion and risk.
