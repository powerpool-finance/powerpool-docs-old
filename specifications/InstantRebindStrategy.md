# InstantRebindStrategy

This strategy updates the composition of the pool by re-allocating pool funds between Yearn Vaults with assigning new weights thereafter. 

The new share and weight of each Vault LP token in a pool correspond to the share of the pool's TVL from a total TVL of all Yearn Vaults LP tokens, included in the pool.

The total supply and capitalization of every Vault LP token directly corresponds to TVL in it, so
this approach can be considered as a sort of M.Cap weighting.

## Formula

```
// Step #1. Compute TVL (USDC equivalent) in each Yearn Vault using the
// corresponding curve depositor pool contract helper (in USDC tokens)
VaultPoolUSD = curveDepositor.calc_withdraw_one_coin(USDC, vaultTotalSupply * vault.getPricePerFullShare())

// Step #2. Estimate the total supply for each Vault LP token. It uses `get_virtual_price_from_lp_token()`
// function since `calc_withdraw_one_coin()` doesn't always work correctly using
// the large out amounts such as vault's totalSupply.
// Additionally, during this step TVLChangeRate is used to multiply TVLWithoutRate for receiving TVL, which is used in denorm weights calculation.
TVLWithoutRate = vaultTotalSupply * vault.getPricePerFullShare() * curvePoolRegistry.get_virtual_price_from_lp_token(vaultToken)
TVLChangeRate = PreviosTVL == 0 ? 1 : TVL / PreviosTVL
TVL = TVLWithoutRate * TVLChangeRate

// Step #3. Compute the new Yearn LP token token denorm weight as share of TotalTSVaultsUSD sum, taking into account totalDenormWeight in this pool
TVLSum = Σ(TVL)
totalDenormWeight = 25
tokenDenormWeight = TVL * totalDenormWeight / TVLSum

// Step #4. Compute the vault's target share
VaultPoolTargetShare = tokenDenormWeight / totalDenormWeight

// Step #5. Compute the target USDC value of the pool
TotalVaultsPoolUSD = Σ(VaultPoolUSD)
TargetUSDCBalanceOnPool = VaultPoolTargetShare * TotalVaultsPoolUSD

// Step #6. Compute the vault USDC/vault token ratio
VaultRate = VaultPoolUSD/VaultPoolTotalSupply

// Step #7. Compute the target vault supply
PoolTargetVaultBalance = TargetUSDCBalanceOnPool / VaultRate
```

## Poke Operation

The actual operations inside the smart contract are optimized for gas efficiency and don't strictly follow the algorithm above. The algorithm of actual operations inside the Poke for instant rebind strategy is presented below:

* Computes the YLA pool total USDC value (USDC out) and caches each vault USDC out. This is required for calculating vault/usdc rate later. / `YearnVaultInstantRebindStrategy.getRebindConfigs()`
* Computes total USDC value for totalSupply for each vault token and their sums. / `WeightValueAbstract.computeWeightsChange()`
* Computes a new weight for token as `newWeight = (vaultUSD / allVaultsUSDC) * fixedTotalWeight` / `WeightValueAbstract.computeWeightsChange()`
* According to new weights, balances of certain Vault LP tokens should be changed (increased or decreased). The balance of Vaults LP tokens which have newWeight > the current one will be increased by adding them to the pool accordingly to the NewWeight. 
  If NewWeight < the current one, the balance of this Vault LP token will be decreased by removing some tokens accordingly to the NewWeight. We sort array of weights for determine which Vaults should be partly removed from the pool/added to the pool. / `WeightValueAbstract.computeWeightsChange()`
* Computes a new balance for each Vault LP token as `newVaultBalance = (totalUSDCPool * tokenDenormWeight / totalDenormWeight) / (poolUSDCBalances[oi] / tokenTotalSupply[oi]))` / `YearnVaultInstantRebindStrategy.getRebindConfigs()`
* Checking each Vault LP token using / `YearnVaultInstantRebindStrategy._instantRebind()`
	* If balance of Vault LP token should be decreased, some amount USDC should be removed the corresponding Vault according to following sequence of operations:
		* Pull the difference oldVaultBalance - newVaultBalance from the pool
		* Unwrap received yVault tokens into crvPool tokens
			* If there were fees charged by the yVault contract, it accrues the fee amount in the corresponding storage slot for further refund. However, since YLA is composed of v2 Vaults, this function isn't used.
		* Unwrap received crvPool tokens into USDC
	* The USDC received from previous operation is used for supplying into other Vaults, which balances should be increased. It works as follows:
		* Compute the virtual USD amount backing the yVault token. There are two ways doing this estimation:
			1. Using the curvePoolRegistry contract's get_virtual_price_from_lp_token logic (in virtual USD tokens)
			2. Using the corresponding curve depositor pool contract helper (in USDC tokens)
			* The outputs of both methods slightly vary. The strategy's owner can switch between these two options anytime.
		* Use the sum of all the virtual USD amounts for supplying liquidity into all yVaults which balances should be increased:
* For each token to pull use: / `YearnVaultInstantRebindStrategy._instantRebind()`
	* Supply a corresponding USDC amount (share) into crvPool token
	* Deposit all the owned crvPool tokens into corresponding yVaults
	* Add all received yVault tokens into the pool
* Checks that the USDC reminder doesn't exceed the value set by the owner. / `YearnVaultInstantRebindStrategy._instantRebind()`


## Example in table

|Current pool balance|Current USDC balance||vaultTotalSupply|Virtual Price|TVLChangeRate|TVLWithoutRate|TVL           |VaultPoolTargetShare|tokenDenormWeight||PoolTargetVaultBalance|TargetUSDCBalanceOnPool|
|--------------------|--------------------|--------------------|----------------|-------------|-------------|--------------|--------------|--------------------|-----------------|-------|----------------------|-----------------------|
|544,083.86          |579,572.04          ||25,220,770.00   |1.065225569  |1.05         |26,865,809.08 |28,209,099.53 |0.1375654899        |3.439137248      |       |567,545.69            |604,564.18             |
|1,609,750.32        |1,649,868.33        ||74,619,274.06   |1.024921884  |1.00         |76,478,926.96 |76,478,926.96 |0.3729598332        |9.323995829      |       |1,599,205.27          |1,639,060.48           |
|784,812.41          |796,636.51          ||36,379,636.91   |1.015066147  |1.00         |36,927,737.86 |36,927,737.86 |0.1800831091        |4.502077728      |       |779,671.31            |791,417.95             |
|898,678.23          |981,047.19          ||41,657,837.38   |1.091655678  |1.00         |45,476,014.72 |45,476,014.72 |0.2217699376        |5.544248441      |       |892,791.22            |974,620.61             |
|353,578.07          |387,613.20          ||16,389,957.26   |1.096259151  |1.00         |17,967,640.63 |17,967,640.63 |0.08762163014       |2.190540754      |       |351,261.87            |385,074.04             |
|                    |4,394,737.27        ||                |             |5.05         |              |205,059,419.69|                    |                 |       |                      |4,394,737.27           |
