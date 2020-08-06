<h1>Power Pool: a solution for accumulating governance power in Ethereum-based protocols</h1>
<h4>Abstract</h4>
<p>Governance tokens have a huge impact on the Ethereum ecosystem playing a key role in the operation of protocols in the billion-dollar Defi industry. Now the minority token holders can extract very limited utility from such tokens for two reasons (1) they cannot really influence the votes (2) the major share of such tokens don’t provide any income. As a result, the fundamental value of such tokens for the minority holder is close to zero and protocols face voters' apathy problem. The Power Pool is a protocol, offering a convenient solution for pooling governance tokens. It allows the token holders to lend, pool, borrow governance tokens, get income from that and accumulate governance power in protocols based on Ethereum. The Power Pool's mission is to expand the utility of governance tokens to the end-users and provide a new level of coordination of decision making in the Defi ecosystem.</p>
<h4>Introduction</h4>
<p>The decision-making is playing a crucial role in everyone’s life. No matter how serious decisions you make, you are personally affected by many choices of other entities like governments, corporations, financial institutions, and many others. Despite the democratic nature of developed societies, all of them de-facto are governed by influencers of different types – corporations, syndicates, and people with political influence and colossal capital.</p>
<p>The crypto industry and, in particular, the DeFi space is considered to be even more democratic and decentralized in decision-making and value distribution. On-chain governance mechanisms are considered as a holy grail to make our industry sustainable and resilient. Recently it became evident that Defi in 2020 de-facto develops under the slogan “everyone gets a governance token” (from now “governance token” = GT). No doubts, a lot of crypto users are involved in dozens of “governance experiments” now with varying degrees of quality. Even third-tier and almost dead scammy ICOs try to become a “Defi startup” and introduce a governance token.</p>
<p>The main difference of nowadays hype from the previous ones (like ICO’s bubble) is that some platforms have dozens of thousands of users and billions of dollars of cumulative locked capital. So, governance questions become especially important when we are talking about vast amounts of money. These tokens are not useless (as they were in the ICO epoch) and definitely play a vital role in the ecosystem. The total capitalization of governance tokens can be assumed as ~3-4 bln USD, and it snowballs. It is possible to make a bold prediction about the $25 bln valuations at the end of 2020.</p>
<p>The main question that arises here is the real monetary value of such tokens for different groups of stakeholders. There is no doubt, generally talking - if there is a significant economic activity based on the protocol, the possibility of governing it is valuable.</p>
<h4>The issues related to governance tokens</h4>
<p>Digging deeper into the design and practical implementations results of governance tokens, some issues are evident:</p>
<ol>
<li>The minority stakeholder (token holder) cannot influence the vote as often whales de-facto run it[1]</li>
<li>The majority of the governance tokens don’t have any cashflows, arriving at a token holder</li>
</ol>
<p>As a result of both mentioned issues, seems that industry as a whole face severe consequences:</p>
<ol>
<li>The fundamental monetary value of such a token for the minority shareholder is close to zero (of course, it is a point for discussion)</li>
<li>Almost 100% of protocols governed by token holders face Voters Apathy (a well-known on-chain governance problem)[2]</li>
</ol>
<h4>An accumulation of governance power and Influence Farming</h4>
<p>We developed an experimental protocol that probably can help to solve mentioned issues providing specials services for the holders of GTs:</p>
<ul>
<li>An accumulation of governance power (by many minority token holders)</li>
<li>Influence Farming</li>
</ul>
<p>According to our vision, it will empower minority token holders, provide an additional utility for GTs and as a result contribute to the robustness and maturity of the Defi ecosystem.</p>
<p>We named it <b>the Power Pool</b>. It allows holders of GTs to:</p>
<ul>
<li>pool GTs - to accumulate governance power (pool the Power!) in one place</li>
<li>lend GTs - to earn additional GTs by lending GTs. The interest rate is paid in the base asset (the governance token itself), so we introduce a new term - <b>Influence Farming</b></li>
<li>borrow GTs - to get additional leverage in votings</li>
<li>coordinate the use of GTs - to use Power Pool as a platform for collecting GTs and voting in coordinated regime</li>
</ul>
<p>In the next sections, we will describe the vision, implementation, and bootstrapping process of the Power Pool.</p>
<h4>Power Pool protocol in a nutshell</h4>
<em>“United we stand, divided we fall”</em>
<p>The PowerPool is a lending protocol for the governance tokens, for example, COMP, BAL, LEND, YFI, BZRX, AKRO, and many others. It is important to note that currently Power Pool is targeted on the Defi market as the hottest one, but generally is not limited to it and can serve for pooling any other governance tokens in the Ethereum ecosystem.</p>
<p>The PowerPool is based on a simple lending model, which is close to Compound’s one from the first view: every holder of GTs can supply liquidity into a contract and get the interest rate for it if there is a demand, and any person on the market can borrow GTs placing allowed digital assets as a collateral. Currently, our vision is to add ETH, wBTC, and DAI as collaterals for borrowing governance tokens. On the other hand, it has certain upgrades and the special set of oracles, developed to form price feeds of highly-volatile assets such as GTs.</p>
<p>Talking about the economic nature of GTs they are definitely unique asset class in the context of lending/borrowing mechanics. Utility of GTs is not constant in time (comparing, for example with payment tokens such as stablecoins). Talking strictly, it appears only during voting. Our vision is to introduce a novel type of lending logic, which is not available by default in Compound or any other lending protocols but can be very suitable for the case of GTs. We plan to cover it in the next articles.</p>
<h4>Features & Advantages</h4>
<p>The Power Pool offers the following advantages to the end-users and Defi protocols:</p>
<ol>
<li><b>Influence farming.</b> Users can earn additional GTs from their holdings. As a result the stake of the user continually grows as the interest rate is paid in the same governance token.</li>
<li><b>Solution for the Voters Apathy.</b> With the yield and LM as incentives, a certain part of the passive token holders probably would be motivated to pool their GTs in the Power Pool. If there is a demand, these tokens will participate in the vote, increasing the overall vote capitalization. If not, the users of the Power Pool decide how to vote, so tokens can participate in the vote even if nobody borrowed them.</li>
<li><b>Accumulation of voting power.</b> The voting power, distributed across the thousands of wallets belonging to minority token holders wasn't used. Now it can be concentrated in one place via the Power Pool and became a real force in governance of protocols.</li>
</ol>
<h4>The protocol token specification</h4>
<p>We introduce a native token of Power Pool protocol - the CVP (Concentrated Voting Power). It will allow the protocol governance (including code upgrades) to be transmitted to the active community and liquidity providers. It is a governance token, which has one brand new function, that is not presented in any Defi protocol, governed by token holders yet.</p>
<p>The token functions (voting on proposals regarding protocol operation):</p>
<ol>
<li>Listing of new liquidity pools in the Power Pool protocol and allocation of liquidity mining rewards to them</li>
<li>Adding and removing collateral types in the protocol</li>
<li>Adding new lending logic to the protocol (now we consider fixed-term agreements and some other ones)</li>
<li>Upgrade and maintain the source code of the smart contracts</li>
</ol>
<p>Additionally, CPV has one special function that wasn’t present in other lending protocols before. It comes down to direct voting using remaining (unclaimed by borrowers) GTs in the Power Pool lending contracts. So, CVP captures value from the possibility to govern the Power Pool protocol itself and from the possibility to decide how pooled GTs will vote.</p>

>The CVP holders decide how unclaimed pooled governance tokens will vote at the every vote occuring in listed Defi protocols
>
>The CVP token doesn't have a premine and has pure liquidity

<p>The token metrics, as well as details of special token functions will be fully featured in the special article coming next days. All you need to know - you cannot buy this token now, there is no sale. If you found something listed at any DEX until the protocol is live in the Mainnet - it is a SCAM. The only option to receive tokens is participation in liquidity mining.</p>
<h4>Roadmap and how to participate?</h4>
<p>At the moment, we have our project ready to be deployed in testnet. Mainnet comes soon. There are three main phases for bootstrapping the protocol:</p>
<ol>
<li>The testnet phase. As our product is experimental, we need to test functionality of main components in the safe testnet environment. We invite all Defi community members to test protocol, find bugs, and try to attack it. It will last approximately one week. Subscribe to our Twitter if you don’t want to miss it! Sweet bounty for active participants is included</li>
<li>The limited mainnet phase. We are engineers, which try to make a secure and reliable product, and we found it very risky to attract a huge amount of liquidity to unaudited protocol. During the limited mainnet phase the Baby version of Power Pool will be launched with a $50,000 liquidity cap. We estimate that the limited phase will last 2 weeks and applications will be open soon.</li>
<li>The unlimited mainnet phase. This phase is uncapped and starts right after passing the security audit and carefully testing all protocol features. From this moment Power Pool will grow, attract more and more liquidity to become a real force in governance of Defi protocols controlled by the community.</li>
</ol>
<p><b>Social Links:</b></p>
<p><b>Discord:</b> https://discord.gg/YDABRVa </p>
<p><b>Twitter:</b> https://twitter.com/powerpoolfi </p>
<h4>Links</h4>
[1] https://cryptoslate.com/makerdao-whale-with-94-voting-power-reduces-dai-stability-fee-by-4 <br>
[2] https://www.evanvanness.com/post/184616403861/aragon-vote-shows-the-perils-of-onchain-governance
<h4>Risk warnings</h4>
<p>It is an experimental software built by Defi enthusiasts on a con-commercial basis. It is highly risky to put your money in the protocol until the security audit is successfully passed. The protocol is decentralized, free to use by everyone and governed directly by its users. The developers, users or any other entity are not responsible for possible effects of its usage, for example affecting the governance and operation of any other decentralized protocols. The CVP is a governance token which is distributed to the end-users of the protocol via liquidity mining and is not associated with any legal entity and doesn’t grant any rights or income. As there was no token sale, the token doesn't have any initially defined price and any deals with it is the responsibility of its holders.</p>
