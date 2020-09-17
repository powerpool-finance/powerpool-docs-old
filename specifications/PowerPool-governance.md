# PowerPool governance specification

This specification describes the general high-level principles of PowerPool protocol governance.
The system was developed according to the following requirements:
* to achieve as high flexibility and modularity as possible;
* to provide the ability to vote to all key PowerPool user groups (CVP holders, Liquidity Providers, vested tokens of Beta and Gamma rounds participants);
* to provide the ability to vote on the main Ethereum network and sidechains to reduce gas costs;
* to provide an additional incentive by rewarding users for participation in voting regardless of their decision.

This document is a working copy that demonstrates the current vision of the team and is subject to change.

## General overview
* `PPGovernor` - the main voting contract, using which the PowerPool community makes decisions about protocol development, sets protocol parameters and upgrades smart contracts. The contract uses several input sources of votes: `CVP Token`, `Vesting "Beta"`, `Vesting "Gamma"`, `LP Mining`,`Voting Boost`. Voting parameters such as threshold (the minimal number of votes required to create a proposal) and quorum (the minimal number of votes required to define passed vote as a valid one) and sources of votes are immutable. The contract may be replaced with a new version at the discretion of the community;
* `CVP Voting Rewards` - this contract distributes CVP rewards between users, voted on the proposal that reached the quorum in proportion to number of casted votes. Users are rewarded for participation in voting (it means that they receive reward regardless of option they voted on and the result of the voting; the main condition for eligibility is that voting was technically valid);
* `xDAi CVP Locker` / `Matic Network CVP Locker` - this category refers to CVP holders who vote using sidechains and deposit CVP in the contract for this purpose. Tokens can be withdrawn at any time and are not transferred to the sidechain. When depositing / withdrawing tokens, information on the number of votes on block is transmitted through the cross-chain bridges and can be used for voting. The contract is voting using the entire CVP token balance;
* `Vesting "Beta"` / `Vesting "Gamma"` - the contract owns vested tokens of the "Beta" and "Gamma" rounds participants and distributes them according to the vesting rules. Unlocked tokens give voting rights in `PPGovernor`. The contract votes using the entire token balance only for one option in voting;
* `LP Mining` - it is a reward contract developed to allows liquidity providers to vote and claim LP rewards. Liquidity providers (Uniswap, Balancer, etc.) can deposit pool tokens and receive a reward in CVP and as well as voting rights, based on stake of these tokens;
* `Voting Boost` - CVP holders can lock CVP tokens in this contract. The longer the tokens are locked, the more votes the holder receives. Tokens are unlocked with the cool down period, this means that after calling the `withdraw()` method, tokens are released according to the release schedule;
* `xDAI Mediator` / `Matic Mediator` - the contract contains information on the number of votes in the sidechain, equivalent to the number of CVP-tokens blocked in `xDAi CVP Locker` / `Matic Network CVP Locker`. The number of votes is used in the `PPVotingL2` contract;
* `PPVotingL2` - the sidechain voting contract. Uses `xDAI Mediator` /` Matic Mediator` as the source for the number of votes per block. Makes a decision on how the balance of tokens of CVP contracts `xDAi CVP Locker` /` Matic Network CVP Locker` votes;
![ppVotingOverview](https://github.com/powerpool-finance/powerpool-docs/blob/master/images/ppVotingOverview.png?raw=true)
## L1 (ETH) Contracts

----
### PPMediatorL1 (ETH)

#### CVP Locking
* allows CVP holders locking/unlocking their CVPs any time;
* in case of deposit/withdraw actions contract sends a message to xDAI Mediator with the holder new balance;
* the contract doesn't cache balance per block/timestamp;
* the proxied calls from xDAI mediator to PPGovernor are strictly limited to:
	* the PPGovernor contract address only;
	* the two following methods only:
		* `castVote(uint proposalId, bool support)`;
		* `propose(address[] memory targets, uint[] memory values, string[] memory signatures, bytes[] memory calldatas, string memory description)`;
* when the ETH Mediator casts a vote on a proposal, the total balance of all the locked CVPs is accounted;
* this contract is upgradeable, proxyOwner role is assigned to a PPGovernor's Timelock;
* the has an owner assigned to a PPGovernorL1's Timelock, the owner can:
	* update AMB address;
	* update PPGovernorL1 address.

```solidity
interface PPMediatorL1 {
	// CVP Holder deposits their CVPs
	function deposit(uint256 amount) external;

	// CVP Holder withdraws their CVPs anytime
	function withdraw(address to, uint256 amount) external;

	// If a AMB message failed to execute anyone can
	// send it again
	function syncBalances(address account[]) external;

	// A call from L2 to create a proposal or cast a vote in PPGovernorL1 contract
	function handleCallGovernorL1(bytes4 _signature, bytes calldata _args) external;

	// PPGovernor sets a new AMB address
	function setAMBAddress(address) external onlyOwner;

	// PPGovernor sets a new PPGovernor address
	function setPPGovernorAddress(address) external onlyOwner;
}
```

#### PPGovernorL1
* the contract a copy of GovernorAlpha contract from Compound V2 repo deployed at
https://etherscan.io/address/0xc0dA01a04C3f3E0be433606045bB7017A7323E38#code with the same
compiler configuration;
* is accompanied by a Timelock contract;
* each instance has unique dependent Timelock contract;
* there will be multiple instances of this contract;
* the only difference with GovernorAlpha is that PPGovernorL1 uses 5 contracts as a source for a user's votes balance:
	* CVP token itself;
	* Vesting contract #1;
	* Vesting contract #2;
	* LP Mining;
	* Voting Boost;
* thus, a CVP holder can use not only their current balance, but an unclaimed vested balance too;
* CVP total supply is hardcoded into the contract;
* guardian is set to 0xB258302C3f209491d604165549079680708581Cc address.

## L2 (xDAI) Contracts

----
### PPMediatorL2 (xDAI)

#### Outgoing calls

* a CvpInterface compatible contract;
* it is used by PPVotingL2 as a source of locked CVP balances;
* it accepts and stores information from AMB about current balances;
* it doesn't account totalSupply value, since the quorum threshold is relative to a predefined total Supply of CVP tokens;
* an external observer could fix this imbalance using a permissionless method of PPMediatorL1 called `syncBalances(address[])`;
* proxies `create a proposal`/`cast a vote` calls from PPVotingL2's Timelock to ETH Mediator;
* contract is proxied, a proxyOwner role is assigned to PPVotingL2;
* contract is ownable; Owner address is assigned to PPVotingL2's Timelock.


```solidity
interface PPMediatorL2 {
	// PPVotingL2 sends createProposal with a given calldata
	function createL1Proposal(bytes calldata) onlyVoting;

	// PPVotingL2 sends castVote with a given calldata
	function castL1Vote(bytes calldata) onlyVoting;

	// A message from PPMediator1 about a changed balance
	function setBalance(address account, uint256 value, uint256 nonce) external onlyBridge;

	// PPVotingL2's Timelock sets a new AMB address
	function setAMBAddress(address) external onlyOwner;
}
```

#### PPGovernorL2

* uses PPMediatorL2 as a source of user balances;
* governorAlpha compatible contract;
* inherits from PPGovernorL1 with only `votingPeriod()` changed to 34560 blocks or ~2 days for 5s blocks;
* is accompanied by a Timelock contract;
* has no owner;
* not proxied;
* guardian is set to 0xB258302C3f209491d604165549079680708581Cc address;
* updates could be done by deploying a new version of a contract with the same PPMediatorL2 as a balances source.

## Common contracts
#### Timelock

* The contract is the exact copy of `Timelock` contract from Compound V2 repo deployed at
https://etherscan.io/address/0x6d903f6003cca6255d85cca4d3b5e5146dc33925#code with a different
 compiler configuration (v0.5.16 instead of v0.5.8).
* It has a corresponding GovernorAlpha-compatible contract with admin permissions

## Common restriction for the aforementioned contracts
* If someone accidentally or intentionally sends ERC20/ERC721 tokens or ETHs,
 they will be permanently locked on these contracts (with exception to CVP tokens at L1AMBMediator);
* all the contracts use the following compiler configuration:
    * Compiler version: v0.5.16+commit.9c3226ce;
    * Optimization: yes/200 runs;
    * EVM version: default (Istanbul).
