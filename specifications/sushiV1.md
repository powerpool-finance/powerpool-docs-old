# Sushi V1 Router

* Compatible with SushiBar V1

* Story #1 example https://github.com/powerpool-finance/powerpool-docs/blob/master/specifications/sushi-story1.csv

* The cases when the xSushi total supply is 0 is not covered, all the txs will revert with 0 division error. The reason is that there is an extremely small chance when all the SushiBar holders including the router contract will unstake their rewards.

* SushiBar has the following deposit/withdraw (enter()/leave()) formulas
	* For deposit `xSushi = Sushi * xSushi total supply / total sushi at sushiBar`
	* For withdrawals `Sushi = xSushi * total sushi at sushiBar / xSushi total supply`

* The Underlying equivalent returned by `getPiEquivalentForUnderlying` for mint/bun operation is always the asked amount:

```
PIa = Ua
```

* Thus, the total amount of Sushi currently available at the SushiBar should be always >= piToken total supply:

```
Ss = xS@Pi * S@Bar / xSts

where:
Ss - stakedSushi
xS@Pi - xSushi balance of piToken
S@Bar - Sushi balance of SushiBar
xSts - xSushi total supply
```

* The staked amount of Sushi (underlyingStaked) is dynamically calculated using the formula:

```
US = PIts - S@Pi
PIts - piToken total supply
S@Pi - SUSHI balance of piToken
```

* The excess of Sushi at the SushiBar could be claimed as a pool reward, by burning some amount of xSushi

```
Rs = S@Pi + Ss - PIts

where:
Rs - pool reward in Sushi tokens
S@Pi - Sushi at piToken
Ss - stakedSushi (see the formula above)
PIts - piToken total supply
```

* Addresses
	* Sushi Token: https://etherscan.io/token/0x6b3595068778dd592e39a122f4f5a5cf09c90fe2
	* SushiBar (xSushi token) https://etherscan.io/address/0x8798249c2e607446efb7ad49ec89dd1865ff4272
