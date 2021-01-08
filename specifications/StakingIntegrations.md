# Underlying Protocols Integrations with Staking

We divide the staking system into `Token wrappers` and their `Routers`:

* `piToken` or `WrappedPiErc20.sol` - a contract that wraps an original `ERC20` token and uses a corresponding `Router` contract for a protocol-specific logic;
* `router` or `PowerIndexRouter` - a protocol-specific controller for a `piToken`.

Every `piToken` has a matching `Router` contract bound to this particular `piToken` using an immutable binding. A `Router` can transfer its managing permissions for a `piToken` to another address, for example, in a `Router` logic upgrade.

* [PiToken](#pitoken)
* [Basic Router](#basic-router)
* [Yearn Router](#yearn-router)
* [Aave Router](#aave-router)

## piToken or WrappedPiErc20

* `piToken` is a wrapper for an ERC20 token;
* initially, there are `piYfi,` `piAave` and `piSushi` wrapper contracts;
* `deposit()`, `withdraw()`, and `pokeRouter()` methods are protected from reentrancy using a shared mutex (OZ.nonReentrant);
* the current `Router` can transfer its permissions anytime to a new router using the `piToken.changeRouter()` method;
* the `Router` has exclusive permissions executing arbitrary external calls on behalf of `piToken` using the `callExternal()` method:
	* if the external call has reverted with a revert reason string, the transaction will also revert with this hijacked reason string;
	* if the external call has reverted without a revert reason string or due `invalid` opcode, the transaction will revert with `REVERTED_WITH_NO_REASON_STRING` reason string;
	* we partially copied the logic from the steps above from AragonOS EVMScriptRunner https://github.com/aragon/aragonOS/blob/4bbe3e96fc5a3aa6340b11ec67e6550029da7af9/contracts/evmscript/EVMScriptRunner.sol#L65;
* when depositing underlying tokens to a piToken, as well when withdrawing underlying tokens, there are two router callbacks called from piToken (see router spec for protocol-specific formulas):
	* `getPiEquivalentForUnderlying` - calculates the resulting `piToken` amount based on underlying token amount;
	* `piTokenCallback` - executes reserve rebalancing logic by staking/redeeming underlying tokens;
* deposit/withdraw methods use the underlying token amount as an argument. To calculate the resulting piToken amount, we use the `getPiEquivalentForUnderlying` method. The piToken/underlying token ratio could be affected by:
	* underlying ERC20 token transfer to the `piToken` contract bypassing deposit callback. It will increase underlying token balance on 'piToken' contract, which increases the underlying amount per a single `piToken`;
	* underlying protocol staking contract (e.g., Aave Safety Module) will slash staked underlying tokens. It will decrease the `piToken` contract balance in the underlying token, which will reduce the underlying amount per a single `piToken;`
* Ignored optimizations:
	* in the `withdraw()` method, it is possible to merge `piTokenCallback` and `getPiEquivalentForUnderlying,` which could save 5K gas. Ignored since the current code easier to read, and the logic replicates the logic from `deposit().`

## Power Index Basic Router

`PowerIndexBasicRouter.sol` is an abstract Router contract. Protocol implementation routers (e.g. `YearnPowerIndexRouter.sol`, `AavePowerIndexRouter.sol,` etc.)  should inherit this contract.

* The following values can be set only once during the contract deployment:
	* `piToken` -  `WrappedPiErc20.sol` address;
	* `pvp` - `PermanentVotingPowerV1.sol` address;
* `PowerIndexBasicRouter.sol` has 2 managing roles: the `Owner` and the `poolRestrictions` grantee checked using `poolRestrictions.isVotingSenderAllowed()` method;
* if the `Owner` sets the rebalancing interval to 0, `piToken` will never bypass rebalancing. Otherwise, `piToken` won't rebalance until this interval has passed from the last rebalance action;
* if the `Owner` sets the reserve ratio to 0, `piToken` will stake all the underlying tokens in the staking contract;
* if the `Owner` sets the reserve ratio to 1, `piToken` will keep all the underlying tokens;
* `claimRewards()` method claims and transfers rewards from the Staking contract (e.g., YfiGovernance, Aave Safety module, Sushi Bar, etc.)  to the `Router.` It's permissionless so that anyone can trigger it anytime;
* `distributeRewards()` method, also permissionless:
	* steps:
		* convert claimed reward to the underlying token (steps varies);
		* if there is a `protocol Fee,` cut off this fee from the converted underlying tokens;
		* wrap the remaining underlying tokens into `piToken`;
		* transfer  wrapped tokens to the pools according to pools current `piToken` balances;
		* execute gulp() on each pool for the distributed `piToken`;
	* will revert if:
		* there are no pools set;
		* there is no pool reward remained after the `protocol Fee` distribution;
		* there is no pool reward remained after wrapping underlying tokens into `piToken`;
		* all the configured pools have zero corresponding `piToken` balance;
		* other constraints depending on implementation.

* The owner has the following permissions:
  * `setVotingAndStaking(address _voting, address _staking)` changes both voting and staking contract addresses. Some `Routers` can have only one of them, so the `Owner` should set the other to the 0 address;
	* `setReserveConfig(uint256 _reserveRatio, uint256 _rebalancingInterval)` updates both `Reserve ratio` and `Rebalancing interval`. Ensures `reserveRatio <= 100%`;
	* `setRewardPools(address[] _rewardPools)` sets a new list of the reward pools. Notice, there is no uniqueness check. It's up to the `Owner` to provide correct values. The `Owner` should set at least one reward pool, but the list could be empty while assigning inside the constructor;
	* `setPvpFee()` updates a `protocol Fee` value, ensures `pvpFee < 100%`.
* Ignored optimizations:
	* `_distributePiRemainderToPools()` - it is possible to cache pool balance in "memory" array, so we don't need 2-nd request. Can save roughly 5K * N gas, where N - number of pools;
* `piTokenCallback()` calls `getReserveStatus()` method before executing stake/redeem actions. Method provides useful information in order to make further decisions on a reserve management and returns the following values:

1.
```
                           / %reserveRatio * (staked + leftOnPiToken - withdrawAmount) \
 expectedReserveAmount =  | ------------------------------------------------------------| + withdrawAmount
                           \                         100%                              /
```
* reserveRationPct - % of a reserve ratio;
* staked - amount of original tokens staked to the staking contract;
* leftOnPiToken - amount of origin tokens left on the piToken (WrappedPiErc20) contract;
* withdrawAmount could be negative in a case of deposit;

2. `status = (reserveAmount > leftOnPiToken) ?  ABOVE                          : BELOW`
3. `diff   = (reserveAmount > leftOnPiToken) ? (reserveAmount - leftOnPiToken) : (leftOnPiToken - reserveAmount)`

## Yearn Router

* Calculates `getPiEquivalentForUnderlying` using the following formula:

```
PIa = Ua * PIts / (BUpi + BUst)

where:
PIa - amount of piTokens to mint/burn within deposit/withdraw actions correspondively;
Ua - underlying token amount to deposit or withdraw;
PIts - piToken total supply;
BUpi - piToken's underlying token balance;
BUst - piToken's yearnGovernance balance;
```

* Stakes underlying YFI to YearnGovernance at 0xBa37B002AbaFDd8E89a1995dA52740bbC013D992
* Receive rewards in yCrv 0xdF5e0e81Dff6FAF3A7e52BA697820c5e32D806A8

* `exit()` operation both claims rewards and withdraws all the staked YFI tokens to the router contract. It won't revert if there is no reward claimed not to prevent exiting;
* `distributeRewards()` operation:
	* Swaps yCRV reward into YFI using the following path:
		* Unwraps yCRV -> USDC using yDeposit at 0xbbc81d23ea2c3ec7e56d39296f0cbb648873a5d3;
		* Swaps USDC -> ETH -> YFI using Uniswap Router at 0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D;
	* will revert if (in addition to the common cases):
		* empty Uniswap swap path in config;
		* 0 YCRV tokens on router;
		* 0 USDC tokens on router after unwrapping at yDeposit;
		* 0 YFI tokens on router after swapping via Uniswap;
		* 0 difference between YFI balance before the opration and after the Unswap swap;
* provides the `Router Owner` with permission to call the following methods on behalf of `piToken`;
	* `stake()` - manually stake;
	* `unstake()` - manually unstake;
	* `setUsdcYfiSwapPath()` - reconfigure Uniswap swap path;
	* `setUniswapRouter()` - reconfigure Uniswap router address;
* provides the `poolRestriction` grantees avialable with `isVotingSenderAllowed()` checks with the following permissions:
	* `callRegister()` - register in voting;
	* `callPropose()` - create a proposal;
	* `callVoteFor()` - vote for a proposal;
	* `callVoteAgainst()` - vote against a proposal;
	* `exit()` - exit with all staked YFI, withdraw rewards to the pools contracts.


## Aave Router

* Calculates `getPiEquivalentForUnderlying` using the following formula:

```
PIa = Ua * PIts / (BUpi + BUst)

where:
PIa - amount of piTokens to mint/burn within deposit/withdraw actions correspondively;
Ua - underlying token amount to deposit or withdraw;
PIts - piToken total supply;
BUpi - piToken's underlying token balance;
BUst - piToken's stakedAave balance;
```

* Stakes underlying AAVE to YearnGovernance at 0x4da27a545c0c5B758a6BA100e3a049001de870f5;
* Receive rewards in AAVE 0x7fc66500c84a76ad7e9c93437bfc5ac33e2ddae9;

* To withdraw funds from staking, `piToken` contract should first call `cooldown(),` wait ten days, and call `redeem().` More details here https://docs.aave.com/developers/protocol-governance/staking-aave#integrating-staking;
* `distributeRewards()` operation:
	* will revert if (in addition to the common cases):
		* 0 AAVE tokens on router;
		* 0 difference between AAVE balance before the opration and after the Unswap swap;
* provides the `router Owner` with permission to call the following methods on behalf of `piToken`:
	* `stake()` - manually stake;
	* `unstake()` - manually unstake;
	* `triggerCooldown()` - manually trigger cooldown;
* provides the poolRestriction grantees avialable with `isVotingSenderAllowed()` checks with the following permissions:
	* `callCreate()` - create a proposal;
	* `callSubmitVote()` - submit for/against vote.
