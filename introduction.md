<h1>Power Pool: a solution for accumulating governance power in Ethereum-based protocols</h1>
<h4>Abstract</h4>
<p>Governance tokens have a massive impact on the Ethereum ecosystem playing a vital role in the operation of protocols in the billion-dollar Defi industry. Now the minority token holders can extract minimal utility from such tokens for two reasons (1) they cannot influence the votes (2) the significant share of such tokens don't provide any income. As a result, the fundamental value of such tokens for the minority holder is close to zero, and protocols face voters' apathy problem. The Power Pool is a protocol, offering a convenient solution for pooling governance tokens. It allows the token holders to lend, pool, borrow governance tokens, get income from that, and accumulate governance power in protocols based on Ethereum. The Power Pool's mission is to expand the utility of governance tokens to the end-users and provide a new level of coordination of decision making in the Defi ecosystem.</p>
<h4>Introduction</h4>
<p>The decision-making is playing a central role in everyone’s life. No matter how serious decisions you make, you are personally affected by many choices of other entities like governments, corporations, financial institutions, and many others. Despite the democratic nature of developed societies, all of them de-facto are governed by influencers of different types – corporations, syndicates, and people with political influence and colossal capital.</p>
<p>The crypto industry and, in particular, the DeFi space is considered to be even more democratic and decentralized in decision-making and value distribution. On-chain governance mechanisms are regarded as a holy grail to make our industry sustainable and resilient. Recently it became evident that Defi in 2020 de-facto develops under the slogan “everyone gets a governance token” (from now “governance token” = GT). No doubts, a lot of crypto users are involved in dozens of “governance experiments” now with varying degrees of quality. Even third-tier and almost dead scammy ICOs try to become a “Defi startup” and introduce a governance token.</p>
<p>The main difference of nowadays hype from the previous ones (like ICO’s bubble) is that some platforms have dozens of thousands of users and billions of dollars of cumulative locked capital. So, governance questions become especially important when we are talking about vast amounts of money. These tokens are not useless (as they were in the ICO epoch) and play a vital role in the ecosystem. As of December 2020, the total capitalization of governance tokens is approximately $20 bln USD and it continues to grow.</p>
<p>The main question that arises here is the real monetary value of such tokens for different groups of stakeholders. If there is a significant economic activity on the protocol, the possibility of governing is valuable.</p>
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
<p>According to our vision, it will empower minority token holders, provide an additional utility for GTs, and as a result, contribute to the robustness and maturity of the Defi ecosystem. </p>
<p>We named it <b>the PowerPool</b>. It allows holders of GTs to:</p>
<ul>
<li>pool GTs - to accumulate governance power (pool the Power!) in one place</li>
<li>lend GTs - to earn additional GTs by lending GTs. The interest rate is paid in the base asset (the governance token itself), so we introduce a new term - <b>Influence Farming</b></li>
<li>borrow GTs - to get additional leverage in votings</li>
<li>coordinate the use of GTs - to use Power Pool as a platform for collecting GTs and voting in coordinated regime</li>
</ul>
<p>In the next sections, we will describe the vision, implementation, and bootstrapping process of the Power Pool.</p>
<h4>Power Pool protocol in a nutshell</h4>

<em>“United we stand, divided we fall”</em>

<p>The PowerPool is a lending protocol for the governance tokens, such as COMP, BAL, LEND, YFI, BZRX, AKRO, and many others. It is important to note that currently, PowerPool is targeted on the Defi market as the hottest one, but generally is not limited to it and can serve for pooling any other governance tokens in the Ethereum ecosystem.</p>
<p>The PowerPool is based on a simple liquidity pool model, similar to Balancer at first sight. Under a liquidity pool model, the voting power of the PowerPool is equal to all of the tokens deposited, maximizing governance power. The pool comprises of the top Defi governance tokens, and additionally includes the CVP governance token. Participants who deposit tokens into the pool will be rewarded in CVP for providing this liquidity. The more a specific token is required to reach target weighting in the index, the higher the CVP yield will be for depositing.</p>
<p>Talking about the economic nature of GTs, they are unique assets in the context of lending/borrowing mechanics. The utility of GTs is not constant in time (comparing, for example, with payment tokens such as stablecoins). Talking strictly, it appears only during voting.</p>
<h4>Features & Advantages</h4>
<p>The Power Pool offers the following advantages to the end-users and Defi protocols:</p>
<ol>
<li><b>Influence farming.</b> By providing Defi tokens, users earn CVP. As the pool grows, the requirement to add CVP to the pool will grow, hence naturally increasing demand for the token.</li>
<li><b>Solution for the Voters Apathy.</b> With the Liquidity Mining as an incentive, passive token holders will be motivated to pool their GTs in the PowerPool. These tokens will participate in voting, increasing the overall vote capitalization.</li>
<li><b>Accumulation of voting power.</b> The voting power, distributed across the thousands of minority token holders is useless. Now it can be concentrated in one place via the PowerPool and become a real force in the governance of protocols.</li>
</ol>
<h4>The protocol token specification</h4>
<p>We introduce a native token of Power Pool protocol - the CVP (Concentrated Voting Power). It will allow the protocol governance (including code upgrades) to be transmitted to the active community and liquidity providers. It is a governance token, which has one brand new function, that is not presented in any Defi protocol, governed by token holders yet.</p>
<p>The token functions (voting on proposals regarding protocol operation):</p>
<ol>
<li>Listing of new liquidity pools in the Power Pool protocol and allocation of liquidity mining rewards to them</li>
<li>Upgrade and maintain the source code of the smart contracts</li>
</ol>
<p>Additionally, CVP captures value from the possibility to govern the Power Pool protocol itself and from the possibility to decide how pooled GTs will vote.</p>

>The CVP holders decide how pooled governance tokens will vote at the every vote occuring in listed Defi protocols
>
>The CVP token doesn't have a pre-mine and has pure liquidity

<h4>Roadmap and how to participate?</h4>
It is best to stay updated with our latest achievements and plans via social media.

<p><b>Social Links:</b></p>
<p><b>Discord:</b> https://discord.gg/YDABRVa </p>
<p><b>Twitter:</b> https://twitter.com/powerpoolcvp </p>
<h4>Links</h4>

[1] https://cryptoslate.com/makerdao-whale-with-94-voting-power-reduces-dai-stability-fee-by-4 <br>
[2] https://www.evanvanness.com/post/184616403861/aragon-vote-shows-the-perils-of-onchain-governance

<h4>Risk warnings</h4>
<p>It is an experimental software built by Defi enthusiasts on a non-commercial basis. It is highly risky to put your money in the protocol until the security audit is successfully passed. The protocol is decentralized, free to use by everyone, and governed directly by its users. The developers, users, or any other entity are not responsible for possible effects of its usage, for example affecting the governance and operation of any other decentralized protocols. The CVP is a governance token distributed to the end-users of the protocol via liquidity mining and is not associated with any legal entity and doesn't grant any rights or income. As there was no token sale, the token doesn't have any initially defined price, and any deals with it are the responsibility of its holders.</p>
