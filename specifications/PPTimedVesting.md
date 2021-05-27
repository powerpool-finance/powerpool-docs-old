# PowerPool Timed Vesting Contract

## Contract purpose
The purpose of this contract is to provide a temporary lock and following it vesting for the `CVP` ERC20 tokens received by participants in the Beta/Gamma testing rounds of PowerPool and PowerPool Core Team. The contract has a special function, which allows one to vote using tokens locked in the contract with additional ability to delegate votes to any address.

## General principles
The contract contains addresses, which should receive `CVP` tokens with lock and linear vesting. During the lock period and subsequent linear vesting, members can use tokens for voting. One vote equals one CVP token. Votes also have linear vesting, the period of which differs from the vesting period of tokens.

## Significant changes from `ppVesting.sol`
- Vesting values are calculated using timestamps instead of block numbers (vote caching still uses block numbers);
- Ownable trait was added back;
- Owner can increase `durationT` value any number of times he wants with a limit of 180 days a time;
- Owner can increase `personalDurationT` for a particular member;
- Owner can disable member anytime;
- A member can renounce his membership anytime;
- A member can delegate votes to a non-member address;


## Initialization

The contract is initialized with the following parameters that can't be changed later:
- token address (immutable);
- pre-defined vote vesting start timestamp `startV` (immutable);
- pre-defined vote vesting duration in seconds `durationV` (immutable);
- pre-defined token vesting start timestamp `startT` (mutable since `PPTimedVesting.sol`);
- the number of votes/tokens, allocated for each member; the same for all members (immutable);
- a list of vesting members (mutable, since it can't be represented as immutable, but there is no way to change it later);

The following params could be changed later by the owner:
- the token vesting duration in seconds `durationT` (mutable since `PPTimedVesting.sol`);

## Specification
- `CVP` tokens for each member are vested linearly between `startT` and `endT.` This means that the number of tokens that address can withdraw from the contract is proportional to the time elapsed since the beginning of the vesting period;
- it is deemed that at the `startT` timestamp, the contract will have `members.length * amountPerMember` tokens on balance. It is a responsibility of a contract deployer to allocate exactly this amount on a contract;
- if a contract deployer allocate insufficient funds, some members won't get their tokens when the vesting period ends;
- if a contract deployer allocates more funds, these funds will be locked in the contract. The same rule applies for the funds accidentally transferred to this contract;
- contract has only one owner at a time;
- the contract uses Ownable trait from OpenZeppelin libs;
- any ERC20/ERC721 tokens or ETHs accidentally sent to the contract will be permanently locked there;
- initialization with Merkle tree root hash, which saves deployment gas usage, was intentionally ignored to simplify contract;
- ERC20 token CAP is 100,000,000 * 10**18 (100000000000000000000000000);
- SafeMath covers calculations;
- compilation configuration: Solidity version: v0.6.12, optimizer: enabled, runs: 200, EVM: Istanbul (we're flexible on compiler version);
- the contract doesn't use a proxy pattern. There is no way to upgrade it;
- there will be multiple instances of this contract with different initialization parameters;
- the contract is compatible with the `CompInterface` and provides cached `getPriorVotes()` information as the `CVP` token does;
- to provide detailed information about the claimed votes at a specific block by a member, each `claimVote()` creates a corresponding vote checkpoint;
- a member has an option to transfer his or her vested tokens and votes to a new address, for example, when a private key is compromised. The new address means that it has never been used as the vesting contract member address before.
- the vote and the token vesting amount per member are equal;
- the vote vesting starts before the token vesting start;
- the vote vesting ends before the token vesting ends;
- the vote vesting and the token vesting durations can be different;
- a member can delegate his already claimed votes to any address he wants;
- an address can't delegate the votes that were delegated to him by other address;
- when member transfers the vesting rights to another address:
  - the remaining unclaimed votes/tokens balances are transferred to the new address;
  - if the member didn't delegate his or her votes to another address (the delegation mapping contains 0 address as a delegate), the adjusted amount migrates to a new address;
  - if the member has delegated votes to another address, the contract transfers these votes to a new address;
  - all the amounts delegated to the initial address remains on this address, so all delegators should explicitly re-delegate their votes;
  - the `claimVotes()` is automatically called on the `migrate to` address;
- before the `endT` timestamp, the `claimTokens()` automatically calls `claimVotes()` to avoid cases when member votes and tokens sums exceed the initial vesting amount;
- after the `endT` timestamp, the `claimTokens()`doesn't call `claimVotes()`;
- after the end of the token vesting for `block.timestamp >= endT,` `getPriorVotes()` function will return `0` for all requests, even for queries against the blocks with non-zero cached votes;
- there are 3 endT values:
  - `endT` - the global endT value
  - `personalEndT` - a custom value for a particular member
  - `memberEndT` - the function which returns:
    - a member's personalEndT if it is not 0
    - the global endT otherwise
- there could be cases when:
  - endT > personalEndT
  - endT == personalEndT
  - endT < personalEndT
- if someone created the proposal before `memberEndT` and a user voted after `personalEndT,` he won't be able to use his votes since the contract will return 0 for `getPriorVotes` after `endT;`
- after the Owner calls `increaseDurationT()` or `increasePersonalDurationT()`, there will be a gap when a user doesn't receive rewards since he has more funds already vested to him than he should receive according to the formula;
- when a member claims votes, the corresponding amounts of votes excluding the already claimed token amount is accrued on delegated voting balance:

```
currentVoteBalance = V * (curr - startV)/(endV - startV) - paidOutTokens

V - total value (tokens/votes) to payout
curr - current timestamp
startV - vote vesting period start timestamp
endV - vote vesting period end timestamp
startT- token versting period start timestamp
memberEndT - either a personal or a global token vesting period end timestamp
```

## Methods

### State changing methods

* claimVotes(address _member) - Anyone can claim currently unlocked vote amount for a given member;
* claimTokens(address _to) - A member itself can claim currently unlocked token amount to the provided address;
* delegate(address _to) - A member delegates his votes to another address;
* transfer(address _to) - A member can transfer the vested right to a new address, for ex. when the initial key is compromised;
* increaseDurationT(uint256 _durationT) - The owner can increase the global endT value
* increasePersonalDurationT(uint256 _personalDurationT) - The owner can increase a member's personal endT value

### View methods

* `function getAvailableVotes(uint256 _alreadyClaimed, uint256 _memberEndT) public view returns (uint256)` - returns the available amount for a vote claim based on the current contract values, an already claimed amount input, and a memberEndT;
* `function getAvailableTokensForMember(address _member) public view returns (uint256)` - returns the available amount for a vote claim by a given member at the current block timestamp based on the current contract values;
* `function getAvailableTokens(uint256 _alreadyClaimed, uint256 _durationT) public view returns (uint256)` - returns the available amount for a token claim based on the current contract values, an already claimed amount input, and a memberDurationT;
* `function getAvailableTokensForMember(address _member) public view returns (uint256)` - returns the available amount for a token claim by a given member at the current block timestamp based on the current contract values;
* `function getAvailableTokensForMemberAt(uint256 _atTimestamp, address _member) external view returns (uint256)` - the same as the above, but provides a value for the given timestamp instead of the current;
* `function getPriorVotes(address account, uint256 blockNumber) external override view returns (uint96)` - provides information about a member unclaimed balance in order to use it in a voting contract;

* `function hasVoteVestingStarted() external view returns (bool)` - checks whether the vote vesting period has started or not;
* `function hasVoteVestingEnded() external view returns (bool)` - checks whether the vote vesting period has ended or not;
* `function hasTokenVestingStarted() external view returns (bool)` - checks whether the token vesting period has started or not;
* `function hasTokenVestingEnded() external view returns (bool)` - checks whether the token vesting period has ended or not;

* `function getVoteUser(_member) external view returns (address)` - returns the address a _member delegated their votes to;
* `function lastCachedVotes(_member) external view returns (256)` - provides debugging information about the last cached votes checkpoint with no other conditions;

### Pure methods
* `function getAvailable(
  uint256 _now,
  uint256 _startTimestamp,
  uint256 _amountPerMember,
  uint256 _durationSeconds,
  uint256 _alreadyClaimed
  ) public pure returns (uint256)` - calculate available per member using the following formula:

```
available = (_now - _startTimestamp) * _amountPerMember / _durationSeconds - _alreadyClaimed;
```
