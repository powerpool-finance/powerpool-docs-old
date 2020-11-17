# PowerPool Vesting Contract V2

## Contract purpose
The purpose of this contract is to provide a temporary lock and following it vesting for the CVP ERC20 tokens received by participants in the Beta and Gamma testing rounds of PowerPool. The contract has a special function, which allows one to vote using tokens locked in the contract.

## Significant changes since V1
- Ownable trait was removed;
- The contract has vote vesting functionality along with token vesting one;
- V1 caches the remainder of unclaimed tokens per block while V2 caches the already claimed votes values per block

## Initialization

The contract is initialized with the following parameters that can't be changed later:
- token address (immutable);
- pre-defined vote vesting start block `startV` (immutable);
- pre-defined vote vesting duration in blocks `durationV` (immutable);
- pre-defined token vesting start block `startT` (immutable);
- pre-defined token vesting duration in blocks `durationT` (immutable);
- the number of votes/tokens, allocated for each member; the same for all members (immutable);
- a list of vesting members (mutable, since it can't be represented as immutable, but there is no way to change it later)

## Specification

- it is deemed that at the `startT` block the contract will have `members.length * amountPerMember` tokens on the balance. It is a responsibility of a contract deployer to allocate exactly this amount on a contract;
- if a contract deployer allocate insufficiently funds some members won't get their tokens when the vesting period ends;
- if a contract deployer allocates more funds that the formula above these funds will be locked in the contract. The same rule applies for the funds accidentally transferred to this contract;
- contact has only one owner at a time;
- the contract doesn't use Ownable pattern;
- any ERC20/ERC721 tokens or ETHs accidentally sent to the contract will be permanently locked there;
- initialization with Merkle tree root hash, which saves deployment gas usage, was intentionally ignored in order to simplify contract;
- ERC20 token CAP is 100,000,000 * 10**18 (100000000000000000000000000);
- calculations will be covered with SafeMath;
- compilation configuration: Solidity version: v0.6.12, optimizer: enabled, runs: 200, EVM: Istanbul (we're flexible on compiler version);
- the contract doesn't use a proxy pattern, there is no way to upgrade it;
- there will be multiple instances of this contract with different initialization parameters;
- the contract is compatible with the following `CompInterface` and provides cached getPriorVotes() information like CVP token does;
- to provide detailed information about the claimed votes at a specific block by a member, each claimVote() creates a corresponding vote checkpoint;
- a member has an option to transfer his or her vested tokens and votes to a new address, for example in a case when a private key is compromised. The new address means that it has never been used as the vesting contract member address before.
- the vote and the token vesting amount per member are equal;
- the vote vesting starts before the token vesting;
- the vote vesting ends before the token vesting ends;
- the vote vesting and the token vesting durations could be different;
- the vote vesting and the token vesting durations could be different;
- a member can delegate his already claimed votes to another member;
- a member can't delegate the votes the other members delegated to him;
- when member transfers the vesting rights to another address:
  - the remaining unclaimed vote/tokens balances are transferred to the new address;
  - if the member has not delegated their votes to someone (the delegation mapping contains 0 address as a delegate), the adjusted amount is migrated to a new address
  - if the member has delegated their votes to another address or to their address explicitly, the delegation is accounted as a new address delegation
  - all the delegated amounts to the initial address remains on this address, so the all delegators should explicitly re-delegate their votes
  - the `claimVotes()` is automatically called on the `migrate to` address
- before the `endV` block the `claimTokens()` call automatically calls `claimVotes()` in order to avoid cases when a member vote and token sum exceeds the initial vesting amount;
- after the `endV` block the `claimTokens()`doesn't call `claimVotes()`
- after the end of the vote vesting for `blockNumber >= endV` all vote balances are returned as 0-s, even if there is some unclaimed votes;
- when a member claims their votes, the corresponding amounts of votes excluding the already claimed token amount is accrued on his or a delegated cached balance:

```
currentVoteBalance = V * (curr - startV)/(endV - startV) - paidOutTokens

V - total value (tokens/votes) to payout
curr - current block
startV - vote vesting period start block
endV - vote vesting period end
startT- token versting period start block
endT - token vesting period end block
```

## Methods

### State changing methods

* claimVotes(address _member) - Anyone can claim currently unlocked vote amount for a given member;
* claimTokens(address _to) - A member itself can claim currently unlocked token amount to the provided address;
* delegate(address _to) - A member delegates his votes to another member;
* transfer(address _to) - A member can transfer the vested right to a new address, for ex. when the initial key is compromised;

### View methods

* `function getAvailableVotes(uint256 _alreadyClaimed) public view returns (uint256)` - returns the available amount for a vote claim based on the current contract values, and an already claimed amount input;
* `function getAvailableTokensForMember(address _member) public view returns (uint256)` - returns the available amount for a vote claim by a given member in the current block based on the current contract values;
* `function getAvailableTokens(uint256 _alreadyClaimed) public view returns (uint256)` - returns the available amount for a token claim based on the current contract values, and an already claimed amount input;
* `function getAvailableTokensForMember(address _member) public view returns (uint256)` - returns the available amount for a token claim by a given member in the current block based on the current contract values;
* `function getAvailableTokensForMemberInTheNextBlock(address _member) external view returns (uint256)` - the same as the above, but provides a value for the next block instead of the current;
* `function getPriorVotes(address account, uint256 blockNumber) external override view returns (uint96)` - provides information about a member unclaimed balance in order to use it in a voting contract;

* `function hasVoteVestingStarted() external view returns (bool)` - checks whether the vote vesting period has started or not;
* `function hasVoteVestingEnded() external view returns (bool)` - checks whether the vote vesting period has ended or not;
* `function hasTokenVestingStarted() external view returns (bool)` - checks whether the token vesting period has started or not;
* `function hasTokenVestingEnded() external view returns (bool)` - checks whether the token vesting period has ended or not;

* `function getVoteUser(_member) external view returns (address)` - returns the address a _member delegated their votes to;
* `function debugLastCachedVotes(_member) external view returns (256)` - provides debugging information about the last cached votes checkpoint with no other conditions;

### Pure methods
* `function getAvailable(
  uint256 _now,
  uint256 _startBlock,
  uint256 _amountPerMember,
  uint256 _durationInBlocks,
  uint256 _alreadyClaimed
  ) public pure returns (uint256)` - calculate available per member using the following formula:

```
available = (_now - _startBlock) * _amountPerMember / _durationInBlocks - _alreadyClaimed;
```

