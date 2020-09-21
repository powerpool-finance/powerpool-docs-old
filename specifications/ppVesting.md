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
- calculations will be covered with SafeMath;
- compilation configuration: Solidity version: v0.6.12, optimizer: enabled, runs: 200, EVM: Istanbul (we're flexible on compiler version);
- the contract doesn't use a proxy pattern, there is no way to upgrade it;
- there will be multiple instances of this contract with different initialization parameters;
- the contract is compatible with the following `CompInterface` and provides cached getPriorVotes() information like CVP token does;
- to provide detailed information about member balance at a specific block, each withdrawal() updates cached value similar to CVP token;
- a member has an option to transfer his or her  vested tokens to another address, for example in a case when a private key is compromised.

## Methods

### State changing methods

* withdraw(address _to) - A member can withdraw currently unlocked amount;
* transferOwnership(address _to) - An owner can transfer his ownership to another address;
* delegateVote(address _to) - An owner can delegate all the balance of this contract to a given address;

### View methods

* `function availableToWithdrawFor(uint256 _alreadyClaimed) public view returns (uint256)` - returns the available amount for withdrawal based on the current contract values and an already claimed amount input;
* `function availableToWithdrawForMember(address _member) public view returns (uint256)` - returns available amount for withdrawal by a given member in the current block based on the current contract values;
* `function availableToWithdrawForMemberInTheNextBlock(address _member) external view returns (uint256)` - the same as the above, but provides a value for the next block instead of the current;
* `function getPriorVotes(address account, uint256 blockNumber) external override view returns (uint96)` - provides information about a member unclaimed balance in order to use it in a voting contract;
* `function hasStarted() external view returns (bool)` - checks whether the vesting period has started or not; 
* `function hasEnded() external view returns (bool)` - checks whether the vesting period has ended or not;

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

interface CompInterface {
    function getPriorVotes(address account, uint blockNumber) external view returns (uint96);
}

contract PPVesting is CompInterface {
    
  struct Member {
    bool active;
    bool transferred;
    uint96 alreadyClaimed;
  }
  
  /// @notice A checkpoint for marking number of votes from a given block
  struct Checkpoint {
    uint32 fromBlock;
    uint96 votes;
  }

  address immutable public token;
  uint256 immutable public startBlock;
  uint256 immutable public durationInBlocks;
  uint96 immutable public amountPerMember;

  address public owner;

  mapping(address => Member) public members;
  /// @notice A record of votes checkpoints for each account, by index
  mapping (address => mapping (uint32 => Checkpoint)) public checkpoints;
  /// @notice The number of checkpoints for each account
  mapping (address => uint32) public numCheckpoints;

  constructor(
    address _tokenAddress,
    uint256 _startBlock,
    uint256 _durationInBlocks,
    address[] memory _membersList,
    uint96 _amountPerMember
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
  
    members[msg.sender].alreadyClaimed += uint96(amount);
    
    _subCache(msg.sender, uint96(amountPerMember - members[msg.sender].alreadyClaimed));
    
    IERC20(token).transfer(_to, amount);
  }

  function transfer(address _to) external {
    Member storage from = members[msg.sender];
    Member storage to = members[_to];

    uint96 alreadyClaimed = from.alreadyClaimed;

    require(from.active == true);
    require(to.transferred == false);
    
    members[msg.sender] = Member({
      active: false,
      transferred: true,
      alreadyClaimed: 0
    });

    members[_to] = Member({
      active: true,
      transferred: false,
      alreadyClaimed: alreadyClaimed
    });
    
    // For ex. amountPerMember = 5000, alreadyClaimed = 2000:
    // 3000 (available) - (5000 amountPerMember - 2000 alreadyClaimed) = 0 newAvailable
    _subCache(msg.sender, amountPerMember - alreadyClaimed);
    // 5000 (available) - 2000 (alreadyClaimed) = 3000 newAvailable
    _subCache(_to, alreadyClaimed);
  }
  
  function _subCache(address _member, uint96 amount) internal {
    uint32 dstRepNum = numCheckpoints[_member];
    uint96 dstRepOld = dstRepNum > 0 ? checkpoints[_member][dstRepNum - 1].votes : uint96(amountPerMember);
    uint96 dstRepNew = sub96(dstRepOld, amount, "Comp::_moveVotes: vote amount overflows");
    _writeCheckpoint(_member, dstRepNum, dstRepOld, dstRepNew);
  }
  
  // The exact copy from COMP token
  function _writeCheckpoint(address delegatee, uint32 nCheckpoints, uint96 oldVotes, uint96 newVotes) internal {
    uint32 blockNumber = safe32(block.number, "Comp::_writeCheckpoint: block number exceeds 32 bits");

    if (nCheckpoints > 0 && checkpoints[delegatee][nCheckpoints - 1].fromBlock == blockNumber) {
      checkpoints[delegatee][nCheckpoints - 1].votes = newVotes;
    } else {
      checkpoints[delegatee][nCheckpoints] = Checkpoint(blockNumber, newVotes);
      numCheckpoints[delegatee] = nCheckpoints + 1;
    }

     emit DelegateVotesChanged(delegatee, oldVotes, newVotes);
  }

  // The exact copy from COMP token
  function safe32(uint n, string memory errorMessage) internal pure returns (uint32) {
    require(n < 2**32, errorMessage);
    return uint32(n);
  }

  // The exact copy from COMP token
  function sub96(uint96 a, uint96 b, string memory errorMessage) internal pure returns (uint96) {
    require(b <= a, errorMessage);
    return a - b;
  }

  // onlyAdmin can trigger this
  function delegateVote(address _to) external {
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
  
  // A copy from COMP token, changes are marked with XXX
  function getPriorVotes(address account, uint blockNumber) public view override returns (uint96) {
    require(blockNumber < block.number, "Comp::getPriorVotes: not yet determined");

    uint32 nCheckpoints = numCheckpoints[account];
    // XXX NEW:
    if (nCheckpoints == 0) {
        if (members[account].active == true) {
            // A member has not claimed any tokens yet
            return uint96(amountPerMember);
        } else {
            // Not a member
            return 0;
        }
    }
    // XXX ORIGINAL
    // if (nCheckpoints == 0) {
    //    return 0;
    // }
    // XXX END 

    // First check most recent balance
    if (checkpoints[account][nCheckpoints - 1].fromBlock <= blockNumber) {
        return checkpoints[account][nCheckpoints - 1].votes;
    }

    // Next check implicit zero balance
    if (checkpoints[account][0].fromBlock > blockNumber) {
        return 0;
    }

    uint32 lower = 0;
    uint32 upper = nCheckpoints - 1;
    while (upper > lower) {
        uint32 center = upper - (upper - lower) / 2; // ceil, avoiding overflow
        Checkpoint memory cp = checkpoints[account][center];
        if (cp.fromBlock == blockNumber) {
            return cp.votes;
        } else if (cp.fromBlock < blockNumber) {
            lower = center;
        } else {
            upper = center - 1;
        }
    }
    return checkpoints[account][lower].votes;
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
