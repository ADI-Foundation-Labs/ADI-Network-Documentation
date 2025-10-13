---
description: Learn about the ADI Network Execution Environment (EE)
---

# Execution Environment

Execution Environment (EE) is ADI's abstraction for a VM. They define their own bytecode format, interpreter, resource consumption and internal frame state. They are initialized and managed by the bootloader. When an EE reaches an exit state (call into another contract, deployment, or return), it yields to the bootloader to handle it.

#### Launch Parameters

An EE needs some initial data to be run. This data includes:

* Resources for execution,
* Bytecode to be run,
* Call information (caller, callee, modifier),
* Call values (calldata, token value).

Execution environment provides a method for creating a new instance (or frame) based on some initial data. Once launched with this data, the EE will execute the bytecode until reaching a _preemption point_.

#### Preemption Points and their Continuation

Preemption points are the possible states on which an EE will yield back to the bootloader.

These are:

* External call request,
* Deployment request,
* External call completed, and
* Deployment completed.

The last two marks indicate that the execution of the EE is complete, and declare the resources returned and the execution result. For the first two, the EE expects the bootloader to do some preparation work (see [Runner Flow](runner-flow.md) for more details) and launch a new EE frame for the call/constructor execution. After this sub-frame is finished, the bootloader will continue the execution of the original EE frame, forwarding the result of the sub-frame.

Thus, the execution environment has to provide methods for the bootloader to continue its execution after a call or deployment request. These Methods need to take back the resources returned by the sub-call, handle its result, and continue executing its own bytecode until reaching another preemption point.

#### EE-specific Functionality for the Bootloader

The execution environment also provides methods that don't involve bytecode execution. Instead, they expose certain information that is specific to the EE, or expose some element of the inner frame state. These include:

* Whether a call modifier is supported,
* Whether the context is static,
* How caller gas should be adjusted before passing it to the callee (think 63/64 rule for EVM),
* How to prepare for a deployment (init code checks, address derivation).
