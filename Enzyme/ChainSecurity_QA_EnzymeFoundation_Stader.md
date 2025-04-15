---
Title:    Extensive QA
Author:   ChainSecurity  
Date:     November 2024
Client:   Enzyme Foundation
---

# Extensive QA

This document reflects an extensive, time-limited QA focused on identifying potential high-impact security vulnerabilities in the code within scope. This QA is not a comprehensive security audit nor does it target finding all vulnerabilities. Instead, it provides a best-effort assessment of whether the code meets core security properties that protect the system’s funds, and highlights areas that may require deeper analysis like a full audit or other security measures.

# Stader Staking Adapter

## Scope

Repository: https://github.com/enzymefinance/protocol

``git diff 297f70c7c60d95e92c97c1f0d0382f884f68f9ec...0e54e714017cd9fae3d9ceaeac68209d326f2dd1`` excluding interfaces and tests.

## Overview

This commit introduces an adapter for Stader. The adapter extends the ``GenericWrappingAdapterBase`` which has already been reviewed. Users of the adapter can only stake ETH by unwrapping an amount of WETH held by the vault and depositing it to Stader.

## Risk

Assuming that the ``GenericWrappingAdapterBase`` is correct, risks only involve unexpected behavior of Stader which could lead to DOSing the adapter.

## Properties Checked

- Correctness of Construction arguments 
- Potential Limitations in the amount to be deposited
- Potential changes in the state Stader

## Issues

INFO: (Theoretical) Stader enforces a deposit limit. The adapter deposits its full balance in ETH to the adapter. Therefore, if someone donated a large amount of ETH to the adapter, the adapter would be DOSed.
NOTE: Stader can change the address of the ETHx token. If the new address is not part of the asset universe of Enzyme then the adapter will not work.


# Limitations and Use of This Report
This report was created at the client’s request and is provided “as-is.” The ChainSecurity Terms of Business apply. All rights reserved. (c) by ChainSecurity.

This QA was performed within a limited scope and timeframe, and cannot guarantee finding all vulnerabilities. We draw attention to the fact that, due to inherent limitations in any software development process, an inherent risk remains that even major failures can remain undetected. The report reflects our assessment of the code at a specific point in time and does not account for subsequent changes to the code, environment, or third-party dependencies. Use of the report and implementation of any recommendations is at the client’s discretion and risk.