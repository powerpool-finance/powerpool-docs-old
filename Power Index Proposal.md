# Power Index Proposal: Team Statement
`A proposal by Delphi Digital was recently published on the Power Forum. It relates to adding a brand new product - a self-balancing GT index into PowerPool protocol. Below is our team statement on the proposal.`

As soon as we learned about this new product idea, our team contacted Delphi Digital and discussed it in detail. Internally, we quickly gave this product a catchy name - the Power Index. In this article we’re publishing a follow-up to our discussions with Delphi Digital, as well as our view on its architecture, development, and implementation into the PowerPool protocol.

The article is divided into four sections:.
- The DeFi Index proposal is devoted to summarizing the key ideas of the Delphi Digital proposal for our community.
- Our opinion on the proposal, team comments and additional findings on this idea.
- The Technical Implementation section is devoted to our plan of delivering it to the market.
- Closing Notes contains some additional remarks, not included in the main text.

All citations in the text are taken from the [original Delphi Digital post](https://gov.powerpool.finance/t/re-imagining-the-powerpool/77/9).

## The Defi Index proposal
As it was published in the original proposal, Delphi Digital suggested the creation of a smart governance token index, consisting of 8 governance tokens, one of which is CVP token. The exact composition of this index has to be defined by the PowerPool community.

The index can be implemented as a kind of Balancer pool (with several major architecture upgrades that we will disclose later in this article), consisting of eight governance tokens. The initial shares of each token are proposed to be ⅛ or 12.5%. All governance tokens that are pooled in this pool will be used in voting in their corresponding protocols based on the decisions of CVP token holders.

Balancer Smart Pools is proposed as a technical solution for this idea due to its flexibility and ability to meet the requirements of the fast-changing DeFi landscape:

`Smart Pools have other attractive features such as adding or removing tokens, pausing swaps, and of course adjustable weights/fees.`

This index is a multipurpose product. It is a web3.0-grade ETF product (the BPT token clearly represents a basket of GTs), it also acts as a liquidity pool for exchanging governance tokens and an instrument to pool a large number of governance tokens into the PowerPool protocol. In addition to that, it provides multiple win-win integration opportunities with other DeFi protocols.

### Bootstrapping and maintenance incentives
Each product, depending on the liquidity supplied by its users, has to be properly bootstrapped and maintained. The basic bootstrapping strategy is based on the fact that there are already a lot of governance token holders:

`Another thing that potentially mitigates this concern is that there are a lot of investors who hold popular DeFi governance tokens like YFI, LEND, SNX, etc. in their portfolios already. Now these investors can deposit them into the pool, receive similar exposure, and yield farm those assets.`

Because of this, the launch of the pool and attraction of liquidity to it is proposed to be based on a CVP liquidity mining strategy. Governance token holders will be able to deposit their tokens into the pool and mine CVP, which has voting power utility at the meta-governance layer.
To maintain shares of governance tokens in this pool, we propose a special feedback loop in the form of dynamic CVP rewards based on the pool composition:
`The PowerPool idea has a reflexive mechanism built in where if the scarce supply and 1/8th target threshold for fees keeps driving the price higher, APY from the liquidity mining rewards also rises with it, potentially allowing us to taper this longer over time.`
It is that simple - if the Power Index has an ideal composition, liquidity mining rewards increase to the maximum possible, and if not - it decreases based on the real pool composition. It is a way to maintain the pool state near the target shares, which were initially defined by the PowerPool community.
The yVault strategy is mentioned as another option to facilitate Power Index bootstrapping. It is noted that yearn.finance is a liquidity router that can supply liquidity to any protocol, where it can be efficiently utilized using yVault:

`Here’s a vault strategy we believe could be attractive for all three parties: YFI holders, PowerPool Liquidity Providers/Farmers, and long-term CVP holders.
Take the PowerPool LP shares which represent a pseudo DeFi ETF and throw it into a vault strategy farming PowerPool’s CVP. Yes, the vault strategy sells the CVP it earns, but you keep the permanent liquidity and the yield from the vault will only keep making it deeper over time also being reflexive.
Thus, the value generated from the vault strategy selling CVP = the amount of new permanent liquidity added to the PowerPool.`

### The all-in-one governance tool
All governance token holders providing assets to this pool will receive additional value as they can not only farm tokens but also participate in governance of different protocols:

`In addition to the CVP rewards they’ll receive, farmers can now take part in fast, cheap governance proposals all in one place across their entire portfolio, boosting engagement that wasn’t there before.`

Taking into account layer-2 integration and the possibility to participate in voting with negligible gas costs, it is unquestionably a very strong and unique feature of the Power Index.
### The permanent voting power treasury
The proposal develops a previously published idea of Delphi initially dubbed a “Blackhole for Governance tokens”. The idea is based on the fact that during a liquidity mining program the main income is based on LM rewards in the form of CVP and not on fees. As the Balancer Pool design includes exchange fees, it is proposed to collect these fees into a special treasury contract, which will hold them (a portfolio containing tokens of the Power Index) infinitely. It will be a portfolio of assets, belonging to a protocol itself representing permanent voting power.

### Current CVP distribution as an advantage

Delphi’s proposal rightly points out that a high, fully diluted CVP market cap can be an advantage to a liquidity mining strategy and possibly can help to bootstrap the protocol:

`The PowerPool idea has a reflexive mechanism built in where, if the scarce supply and 1/8th target threshold for fees keeps driving the price higher, APY from the liquidity mining rewards also rises with it, potentially allowing us to taper this longer over time.`

### Final notes: strong integrations and the future family of Power Pools
As it was proposed by Delphi, Power Index can be just the first step to build a whole ecosystem of different indexes consisting of governance tokens. If the first Power Index gains sufficient traction, indexes for other sectors of the market can be released.

The final part of the proposal is devoted to highlighting the necessity of strong integrations with other Defi products to deliver it to the market in the proper way. The ongoing PowerPool integrations with xDAI and Boardroom have been mentioned.

`More integrations are necessary though to really help PowerPool gain enough traction to make the move from exciting experiment to an innovative project with a lasting impact on the space.
An integration with exchange aggregators such as 1inch, Debank, and Dex.Ag could end up funneling some meaningful volume to PowerPool if its pools attract the depth some of these farming initiatives have had in the past.`

## Our view on this proposal and discussion
First of all, our team is grateful for such an impressive idea and all discussions and brainstorming that was made to polish, integrate it into the PowerPool technical roadmap, and create an efficient bootstrapping strategy based on the current market state.

The proposal explores a lot of small, but critically important details. It discusses the Power Index from different sides, highlighting incentives, bootstrapping strategy, value proposition, and key recommendations to make it successful. From our point of view, it can become a core PowerPool product when properly implemented.

We see it as a complex product, targeting various audiences of Defi users. From the one side, it is a classic financial ETF product that can be used for investors and traders to seamlessly operate with a basket of assets. From the other side, it is a very solid solution for performing the mission of PowerPool - pool governance tokens and establish a meta-governance layer within web3.0. Other use cases include a market for exchanging governance tokens for one another with deep liquidity, yVault-style fund management strategies, and usage shares of pools (BPT tokens) as governance token-based collateral in various Defi products. Without a doubt, there are other applications for this solution - in this article, we only discuss a few of them.

It is important to note that the necessary incentives to maintain this pool were proposed. We are on the same page with Delphi on this point - it is important not only to launch a product but also to maintain it effectively from the first day of operation.

Power Index is a community-driven product from its origination in the form of the governance forum proposal. The pool composition (which tokens will be initially formed this index) as well as its future changes will be based on CVP holders' decisions. So, it will be a flexible solution, which meets community demand to rearrange the token set or share of particular assets in this pool. The technical implementation of the product and the bootstrapping process will be discussed in the next section.

Also, we want to note that implementation of this product doesn’t mean that we will close operation of our GT Money Market (the solution, tested in Alpha, Beta and Gamma rounds, available on Ethereum mainnet and two L2 networks). We see a good synergetic effect for both products here, and definitely will add the BPT token of this pool (the Power Index ETF share, talking in financial terms) to the PowerPool money market. So, liquidity suppliers of Power Index and all BPT token holders will be able to lend these tokens or take a loan using them as a collateral.

One of the biggest features of PowerPool is that it is a meta-governance protocol, collecting GTs, majority GT holders, minority GT holders, and Protocol Politicians in one place and providing the necessary tools to plan and execute decisions across the protocol. Talking about Power Index governance, it perfectly fits the modular nature of the actual PowerPool governance system.

The PowerPool governance system can use different sources of votes: CVP tokens, BPT/UniV2 LP tokens, locked tokens in the Beta/Gamma vesting contracts, Vote Boost contract votes (it is the special contract, where user can lock his tokens and get an additional voting power for that), xDAI and Matic root contracts (it allows users to vote using corresponding L2 networks and save on gas costs). So, all protocol users can vote in a convenient way, providing liquidity to Balancer/Uniswap pools, being Beta/Gamma testers or users of L2 networks. Launching Power Index, we will add a BPT token of this pool as an additional source of votes, so all Power Index liquidity providers will be able to vote in PowerPool governance.

## Technical implementation and product roadmap
To build Power Index with all necessary functions (such as a possibility to vote in Defi protocols using pooled tokens and change pool composition on community demand) our team forked Balancer and implemented all necessary changes to it. It also includes a brand new UI/UX, which will fit the functionality of the pool.

Using this fork, it is easy to launch new Index Pools and also establish additional trading pairs for CVP and other GTs on the platform, if it will be demanded by the PowerPool community.

Launch of the Power Index is planned to execute in several phases. On the Phase 0, we await a proposal from Delphi regarding the Power Index launch. At Phase 1, the initial composition of the pool (talking in the simple language - which tokens will be in) will be established. During Phase 1, PowerPool team will launch a small “test” Baby Power Index in the Ethereum mainnet to demonstrate this product to the community and its integrations with our governance system. It will be capped, but publicly available to everyone. If the community decides to allocate certain CVP rewards for participation in pool testing, the activity of testing this pool will be rewarded.

If this idea will be initially accepted by PowerPool community, the liquidity mining program for Power Index will be designed by mutual efforts of PowerPool, Delphi Digital, and any other interested parties (we wait for all participants in brainstorming on our Power Forum in the special section, which will be launched soon).

Finally, once we will receive the results of the security audit of the Balancer fork (Power Balancer, as we defined it), it will be uncapped and open for liquidity provision and mining CVP tokens. As the PowerPool team, we will make a call to all our supporters and protocol enthusiasts to share their ideas for integration of Power Index with any other Defi instruments and protocols.

## Closing notes
As a closing note, we want to point out that PowerPool is a community-driven project so will Power Index exist or not is based on CVP token holders decision. This also applies to the liquidity mining program and all other activities, ongoing on product testing and launch. Please, join our Power Forum, and share your ideas there, create proposals and help us to build the meta-governance protocol for web3.0.
