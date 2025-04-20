---
layout: post
title: "Timelock contracts"
date: 2025-4-20 08:06:58+08:00
categories: smart contract
---
# Summary 

* [The basis](#the-basis)
* [Good reasons to implement a timelock](#good-reasons-to-implement-a-timelock)
* [Terminology](#terminology)
* [Operation struct](#operation-struct)
* [Operation lifecycle](#operation-lifecycle)
* [Roles](#roles)
* [A Timelock contract with OpenZeppelin](#a-timelock-contract-with-openzeppelin)

# The basis

1. A Timelock is a smart contract that delays function calls of another smart contract after a predetermined amount of time has passed

* **Transaction Scheduling** - A function call or transfer is queued with a release time.

* **Time validation** - Before execution, the contract checks wether the current time(`block.timestamp`) is greater than or equal to the release time.

* **Execution** - Once the release time is reached, the operation is executed.

# Good reasons to implement a timelock

1. Project owner give up control to give more provide more guarentees to the community

2. Timelocks allow investors to exit the protocol in time if neccessery

3. Project admins required to be transparent with stakeholders/investors

# Terminology

* **Operation:** A transaction(or a set of transaction) that is the subject of the timelock. It has to be scheduled by a proposer and executed by a executor. The timelock enforces a minimum delay between the proposition and the execution(see operation lifecircle). If the operation contains multiple transactions(batch mode), they are executed atomically, Operations are identified by the hash of their content.

* **Operation status:**

    * **Unset:** An operation that is not a part of timelock mechanism.
    
    * **Pending:** An operation that has scheduled, before the timer expires.

    * **Ready:** An operation that has been scheduled, ater the timer expires.

    * **Done:** An operation that has been executed.

* **Predecessor:** An (optional) dependency between operations. An operation can depend on another operation(its predecessor), forcing the execution order of these two operations.

* **Roles:** 

    * **Admin:** An address(smart account or EOA) that is in charge of granting the roles of Proposer and Executor.

    * **Proposer:** An address(smart account or EOA) that is in charge of secheduling (and canceling) operations.

    * **Executor:** An address(smart account or EOA) that is in charge of executing operations once the timelock has expired. The role can be given to the zero address to allow anyone to execute operations.

# Operation struct

Operation executed by the [TimelockController](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController) can contain one or multiple subsequent calls.Depending on whether you need to multiple calls to be executed atomically, you can eighter use single or batched operations.

Both operations contains:

* **Target,** the address of the smart contract that the timelock should operation on.

* **Value,** in wei, that should be sent with the transaction. Most of time this will be 0. Ether can be deposited before-end or passed along when executing the transaction.

* **Data,** containing the encoded function selector and parameters of the call. This can be produced using a number of tools. For example, a maintenance operation granting role `ROLE` to `ACCOUNT` can be encoded using web3js as follows:

```
const data = timelock.contract.methods.grantRole(ROLE, ACCOUNT).encodeABI()
```

* **Predecessor,** that specifies a dependency between operations. This dependency is optional. Use `bytes32[0]` if the operation does not have any dependency.

* **Salt,** used to disambiguate two otherwise identical operations. This can be any random value.

In the case of batched operations, `target`,`value` and `data` are specified as arrays, which must be of the same lenghth.

# Operation lifecycle

Timelocked operations are identified by a unique id(their hash) and follow a specific lifecycle:

`Unset`->`Pending`->`Pending`->`Ready`->`Done`

* By calling [schedule](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-schedule-address-uint256-bytes-bytes32-bytes32-uint256-)(or [scheduleBatch](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-scheduleBatch-address---uint256---bytes---bytes32-bytes32-uint256-)), a proposer moves from the operation from the `Unset` to the `Pending` state. This starts a timer that must be longer than the minimum delay. The timer expires at a timestamp accessible throgh the [getTimestamp](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-getTimestamp-bytes32-) method.

* Once the timer expires, the operation automatically gets the `Ready` state. At this point, this can be executed.

* By calling [execute](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-TimelockController-execute-address-uint256-bytes-bytes32-bytes32-) (or [executeBatch](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-executeBatch-address---uint256---bytes---bytes32-bytes32-)), an executor triggers the operation's underlying trnasactions and moves it to the `Done` state.If the operation has predecessor, it has to be in the `Done` state for this transition to succeed.

* [Cancle](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-TimelockController-cancel-bytes32-) allows proposers to cancel any `Pending` state operation.This resets the operation to `Unset` state. it is thus possible for a proposer to re-schedule an operation that has been cancelled. In this case, the timer restarts when the operation is re-scheduled.

Operations status can be queried using the fucntions:

* [isOperationPending(bytes32)](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-isOperationPending-bytes32-)

* [isOperationReady(bytes32)](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-isOperationReady-bytes32-) 

* [isOperationDone(bytes32)](https://docs.openzeppelin.com/contracts/4.x/api/governance#TimelockController-isOperationDone-bytes32-)

# Roles

## Amdin

The admin are in charge of proposers and executors. For the timelock to be self-governed, the should only be given to the timelock itself. Upon deployment, the admin role can be granted to any address(in addition to the timelock itself). After further configuration and testting, this optional admin should renounce its role such that all further maintenance operations have to go through the timelock process.

This role is identified by the **TIMELOCK_ADMIN_ROLE** value:
`0x5f58e3a2316349923ce3780f8d587db2d72378aed66a8261c916544fa6846ca5`

## Proposer

The proposers are in charge of scheduling (and cancelling) operations. This is a critical role, that should be given to governing entities. This could be an EOA, a mulsig, or a DAO.

> **Warning**
>
> **Proposer fight:** Having multiple proposers, while providing redundancy in case one becomes unavaiable, can be dangerous. As proposer have their say on all operations, they could cancel operations they disagree with, including operations to remove them from the proposers.

The role is identified by the **PROPOSER_ROLE** value:
`0xb09aa5aeb3702cfd50b6b62bc4532604938f21248a27a1d5ca736082b6819cc1`

## Executor

The executors are in charge of executing the operations scheduled by proposers once the timelock expires. Logic dictates that mulsig or DAO that are proposers should also be executors in order to guarantee operations that have been scheduled will eventually be executed. However, having additional executors can reduce the cost(the executing transaction does not require validation by the mulsig or DAO that proposed it), while ensure whoever is in charge of execution cannot trigger actions that have not been scheduled by the proposers. Alternatively, it is possible to allow *any* address to execute a proposal once the timelock has expired by granting the exeutor role to the zero address.

This role is identified by the **EXECUTE_ROLE** value:
`0xd8aa0f3194971a2a116679f7c2090f6939c8d4e01a2a8d7e41d55e5351469e63`

> **Warning**
>
> A live contract without at least one proposer and one executor is locked. Make sure these roles are filled by reliable entities before the deployer renounces its administrative rights in favour of the timelock contract itself.See the [AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl) documentation to learn more about role management.


# Reference
[Protect Your Users With Smart Contract Timelocks](https://blog.openzeppelin.com/protect-your-users-with-smart-contract-timelocks)

[Timelock Contracts and DeFi Security: Lessons from Compound](https://medium.com/@srinivasjoshi66/timelock-contracts-and-defi-security-lessons-from-compound-fe24f3e4574b)

[Timelock Terminology](https://docs.openzeppelin.com/contracts/4.x/api/governance#timelock-terminology)
