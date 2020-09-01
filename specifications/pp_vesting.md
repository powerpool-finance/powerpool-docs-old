# PowerPool Vesting Contract

## Contract purpose
The purpose of this contract is to provide a temporary lock and following it vesting for the CVP ERC20 tokens received by participants in the Beta and Gamma testing rounds of PowerPool. The contract has a special function, which allows one to vote using tokens locked in the contract.

## Initialization

The contract is initialized with the following parameters that can't be changed later:
- token address (immutable);
- pre-defined startBlock (immutable);
- pre-defined durationInBlocks (immutable);
- the number of tokens, allocated for each member; the same for all members (immutable)
- a list of vesting members (mutable, since it can't be represented as immutable, but there is no way to change it later) and a mutable owner address.

## Specification

- it is deemed that at the `startBlock` the contract will have `members.length * amountPerMember` tokens on the balance. It is a responsibility of a contract deployer to allocate exactly this amount on a contract;
- if a contract deployer allocate insufficiently funds some members won't get their tokens when the vesting period ends;
- if a contract deployer allocates more funds that the formula above these funds will be locked in the contract. The same rule applies for the funds accidentally transferred to this contract;
- contact has only one owner at a time;
- an owner can transfer his ownership to another account immediately without timelock;
- any ERC20/ERC721 tokens or ETHs accidentally sent to the contract will be permanently locked there;
- initialization with Merkle tree root hash, which saves deployment gas usage, was intentionally ignored in order to simplify contract;
- ERC20 token CAP is 100,000,000 * 10**18 (100000000000000000000000000);
- Calculations will be covered with SafeMath;
- Compilation configuration: Solidity version: v0.6.12, optimizer: enabled, runs: 20000, EVM: Istanbul (we're flexible on compiler version);
- The contract doesn't use a proxy pattern, there is no way to upgrade it;
- There will be multiple instances of this contract with different initialization parameters;

## Methods

### State changing methods

* withdraw(address _to) - A member can withdraw currently unlocked amount;
* transferOwnership(address _to) - An owner can transfer his ownership to another address;
* delegateVote(address _to) - An owner can delegate all the balance of this contract to a given address;

### View methods

* function availableToWithdrawForMember(address _member) public view returns (uint256);
* function availableToWithdrawForMemberWithCheck(address _member) public view returns (uint256) - the same as above,but returns 0 if a member is not active;

### Pure methods
* function availableToWithdraw(
      uint256 _now,
      uint256 _startBlock,
      uint256 _amountPerMember,
      uint256 _durationInBlocks,
      uint256 _alreadyClaimed
  ) public pure returns (uint256) - calculate available per member using the following formula:

```
available = (_now - _startBlock) * _amountPerMember / _durationInBlocks - _alreadyClaimed;
```

## Prototype

```solidity
pragma solidity 0.6.12;

interface IERC20 {
  function transfer(address _to, uint256 _amount) external;
}

interface ICRV {
  function delegate(address delegatee) external;
}

contract PPVesting {

  struct Member {
    bool active;
    uint128 alreadyClaimed;
  }

  address immutable public token;
  uint256 immutable public startBlock;
  uint256 immutable public durationInBlocks;
  uint256 immutable public amountPerMember;

  address public owner;

  mapping(address => Member) public members;

  constructor(
    address _tokenAddress,
    uint256 _startBlock,
    uint256 _durationInBlocks,
    address[] memory _membersList,
    uint256 _amountPerMember
  ) public {
    token = _tokenAddress;
    owner = msg.sender;

    amountPerMember = _amountPerMember;
    startBlock = _startBlock;
    durationInBlocks = _durationInBlocks;

    uint256 len = _membersList.length;

    for (uint256 i = 0; i < len; i++) {
        members[_membersList[i]].active = true;
    }
  }

  function withdraw(address _to) external {
    require(members[msg.sender].active == true);

    uint256 amount = availableToWithdrawForMember(msg.sender);
    require(amount > 0);

    members[msg.sender].alreadyClaimed += uint128(amount);

    token.transfer(msg.sender, amount);
  }

  function delegateVote(address _to) external onlyAdmin {
    ICRV(token).delegate(_to);
  }

  // GETTERS
  // will return amountPerMember for non members, but this check will be covered with an extra function
  function availableToWithdrawForMember(address _member) public view returns (uint256) {
    Member storage member = members[_member];

    return availableToWithdraw(
        block.timestamp,
        startBlock,
        amountPerMember,
        durationInBlocks,
        member.alreadyClaimed
    );
  }

  // PURE
  function availableToWithdraw(
      uint256 _now,
      uint256 _startBlock,
      uint256 _amountPerMember,
      uint256 _durationInBlocks,
      uint256 _alreadyClaimed
  ) public pure returns (uint256) {
    return (_now - _startBlock) * _amountPerMember / _durationInBlocks - _alreadyClaimed;
  }
}
```
