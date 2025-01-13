---
layout: post
title: "Upgradeable solidity contract"
date: 2025-1-11 08:16:58+08:00
categories: smart contract
---
# Summary 

* [The basis](#The basis)
* [A few minor caveats](#A few minor caveats)
* [A proxy contract with OpenZeppelin](#A proxy contract with OpenZeppelin)

# The basis
1. Proxy contracts use `delegatecall` to call the implementation contract
2. Storage is existed in the proxy contract
3. Implementation contract's address is stored in storage slot `0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc`(obtained as `bytes32(uint256(keccak256('eip1967.proxy.implementation')) - 1)`), 
Admin address is stored in storage slot `0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103`(obtained as `bytes32(uint256(keccak256('eip1967.proxy.admin')) - 1)` [EIP-1967](https://eips.ethereum.org/EIPS/eip-1967)

# A few minor caveats
1. Always extend storage instead of modifying it
2. Make sure your contracts use initializer functions instead of constructors

# A proxy contract with OpenZeppelin

Design a contract:
1. Upgradeable
2. Record address and tokens

[BankV1 implement transaction](https://sepolia.etherscan.io/tx/0xaf75611792d0a7c328ef063b6a22c9943a995eb297c51b7b53c2b1e875e643d8)

[BankV1 implement source code](https://sepolia.etherscan.io/address/0x9a814ae4af72695a48f358c414d45123faac15b8#code)

[Proxy contract creatation](https://sepolia.etherscan.io/tx/0x214c3d5e099b36b940c17248ece848e2be553178440a82eab132d30110de2769)

[BankV2 implement transaction](https://sepolia.etherscan.io/tx/0x89b65b42d4791ba1d4507ad9cbb25ff74344d88f3bf490981efdc3e2fb871246)

[BankV2 implement source code](https://sepolia.etherscan.io/address/0x5f1fd823c3ffc2c41578a940212a8cc57bfa11fe#code)

[Bank upgrade transaction](https://sepolia.etherscan.io/tx/0x9f5cd4a8acca3d2699285ef2e7578858516a71da93d216f0e375d1b8c414b104)

# eth_getStorageAt
Contracts' storage layouts 
```
mapping(address => uint256) internal balances;
address public owner;
uint256 public withdrawalFee;
```
Storage layouts should be
* balances
0. 0x43Db1155C06548666E2928f4970694CA21B1835a 0.0002 eth
1. 0xcfc247B51f02d415C3490c1652426d9174201a73 0.0003 eth
* owner 0x43Db1155C06548666E2928f4970694CA21B1835a 
* withdrawalFee 1

## For simple Variables: Stored sequentially in slots.
```
cast storage 0x00CEDc793A6a48eb9e0bE53D79d9e983ab96A5E8 0x01
```
We got 
```
0x00000000000000000000000043db1155c06548666e2928f4970694ca21b1835a
```

## For mappings, the storage location is calculated using:
```
keccak256(key.slotIndex)
```
We got storage slot for an entry in a mapping

```
cast index address 0xcfc247B51f02d415C3490c1652426d9174201a73 0
-> 0xe4e0f75d5e229d70c237640257432bc6e8aaeb17d705a1380bbf7b2516008d10
```

Let's get the map value 
```
cast storage 0x00CEDc793A6a48eb9e0bE53D79d9e983ab96A5E8 0xe4e0f75d5e229d70c237640257432bc6e8aaeb17d705a1380bbf7b2516008d10
-> 0x000000000000000000000000000000000000000000000000000110d9316ec000
cast to-dec 0x000000000000000000000000000000000000000000000000000110d9316ec000
-> 300000000000000
```

# Reference
[Writing Upgradeable Contracts](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable)
[Proxy Upgradeable Pattern](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable)
