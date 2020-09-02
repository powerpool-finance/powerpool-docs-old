# Power Pool Voting Contract

The Power Pool protocol is governed and upgraded by CVP token-holders, using three distinct components; the CVP token, governance modules (PPVoting), and Timelock. Together, these contracts allow the community to propose, vote and implement changes through the administrative functions of a ppToken or the Comptroller. 
Proposals can include changes like adjusting an interest rate model, adding support for a new asset, and also, vote on behalf of Governance tokens (COMP, LEND, YFI, etc.) blocked in pools.

There are two types of CVP tokens - unlocked and locked. Unlocked tokens are transferable tokens, which are owned by CVP holders. Locked CVP tokens are held by Vesting contracts and unlocked linearly over time. 
Holders of locked tokens can participate in protocol governance using second-level voting contracts(Voting L2 A and Voting L2 B). Second layer voting contracts can create proposals and vote in the main governance contract(Voting L1).

This document describes two contracts:
* `PPVoting` which is a fork of `GovernorAlpha`;
* `Timelock` owned by `PPVoting` and responsible for executing enqueued transactions;

## PPVoting Specification

* The contract is the exact copy of GovernorAlpha contract from Compound V2 repo deployed at
https://etherscan.io/address/0xc0dA01a04C3f3E0be433606045bB7017A7323E38#code with the same
compiler configuration:
    * Compiler version: v0.5.16+commit.9c3226ce
    * Optimization: yes/200 runs
    * EVM version: default (Istanbul).
* Each instance has a unique dependent Timelock contract.
* There will be multiple instances of this contract.

## Timelock Specification

* The contract the exact copy of `Timelock` contract from Compound V2 repo deployed at
https://etherscan.io/address/0x6d903f6003cca6255d85cca4d3b5e5146dc33925#code with the same
compiler configuration:
    * Compiler version: v0.5.8+commit.23d335f2
    * Optimization: yes/ 200 runs
    * EVM version: default (Petersburg).
* There will be multiple instances of this contract.

## Initialization

#### Voting L1

There are 3 following parameters for a contract initialization:
* timelock: a corresponding unique instance of `Timelock` contract with delay of 172800;
* comp: address of CVP token;
* guardian: 0x0 (zero) address;

#### Voting L2 A

There are 3 following parameters for a contract initialization:
* timelock: a corresponding unique instance of `Timelock` contract with delay of 172800;
* cvpVesting: address of corresponding PPVesting A contract, which should be compatible with `CvpInterface`;
* guardian: 0x0 (zero) address;

#### Voting L2 B

There are 3 following parameters for a contract initialization:
* timelock: a corresponding unique instance of `Timelock` contract with delay of 172800;
* cvpVesting: address of corresponding PPVesting B contract, which should be compatible with `CvpInterface`;
* guardian: 0x0 (zero) address;

## Expected deployment configuration
```
                     +-------------+
                     |             |
        +----------->+  Voting L1  +<------------+
        |            |             |             |
        |            +-------------+             |
        |                                        |
+-------+---------+                   +----------+------+
|                 |                   |                 |
|  Timelock  A    |                   |  Timelock  B    |
|                 |                   |                 |
+-------+---------+                   +----------+------+
        ^                                        ^
        |                                        |
        |                                        |
+-------+---------+                   +----------+------+
|                 |                   |                 |
|  Voting L2 A    |                   |  Voting L2 B    |
|                 |                   |                 |
+-------+---------+                   +----------+------+
        |                                        |
        | getPriorVotes()                        | getPriorVotes()
        v                                        v
+-------+---------+                   +----------+------+
|                 |                   |                 |
|  Vesting   A    |                   |  Vesting   B    |
|                 |                   |                 |
+-----------------+                   +-----------------+

```
