# Synthetix model

### Overview

I wanted to dig into synthetic assets on DeFi and landed on Synthetix, unsurprisingly. Synthetic assets provide benefits like avoiding significant slippage on AMM-based DEXes when you trade, and providing exposure to assets from the real world (e.g. foreign exchange, commodities).

While trying to understand how the Synthetix model works, I discovered that the system is more complex that I am used to from a tradfi lens. This is partly because the system _is_ complex. After many hours of research, I realized the confusion/complexity also comes from inadequate explanation. The official docs and community blogs do well at explaining the high level stuff, such as how you could buy something that represents direct exposure to the price of bitcoin, but are pretty light on explaining how the whole system is held together mathematically.

When reading about how Synthetics enables exposure to crypto without holding the underlying, and without paying hefty slippage costs on AMMs, one could easily be misled to think that taking part in Synthetix staking automatically represents participation in the global pool of synthetic products ("Synths"), meaning believing that one is long crypto. Is that true? How would that work?

Or, one could buy sBTC, a Synth that represents price exposure to bitcoin. If the price of bitcoin spikes, when should I worry that my counterparty (effectively the SNX stakers) are able to meet their obligations so that the whole system stays solvent and I can capture my profits?

The explanations available online didn't feel complete in light of these questions. I didn't want to take the blog articles at face value, so I modeled out the economics behind Synthetix. Specifically, I focused on the outcomes for an SNX staker.

***Takeaway: My model illustrates how if you stake SNX and simply hold on to your sUSD, you are effectively short the pool of Synths.*** When the value of the debt pool's component Synths rises, you suffer a loss and may have to re-collateralize towards the 600% collateralization ratio. To protect yourself against this scenario, you need to exchange your minted sUSD for other Synths in the Synth pool (aka the "debt pool"), in the same ratio that they exist in the pool. For example, if there are $1 million worth of sUSD and $3 million worth of sBTC in the pool, and I staked SNX to mint $10,000 worth of sUSD, I would convert $7,500 into sBTC. In this way, when the price of bitcoin changes, you do not suffer a loss (or gain).

__However, you would still on the hook for depositing more collateral if the value of the Synth pool rises and makes your collateralization ratio fall below 600%. Some related formulae:__
  __* Collateralization ratio = user's SNX staked in Synthetix / user's share of the debt pool aka Debt Ratio__
  __* Debt Ratio = (new sUSD minted from staking SNX / size of Synth pool at the time of minting sUSD) + incremental changes to the debt ratio based on changes in the debt pool arising from more sUSD minters and changes in the price of Synths__
  __* Incremental changes to the debt ratio for other stakers based on changes in the debt pool aka "delta" = new mint / (existing debt + new mint).)__
____

### Stating the unstated complexity

While working in tradfi I learnt that synthetic financial products are created in one of at least two ways:

**(1) By combining instruments**

In this approach, a trader would combine a series of financial instruments that on a net basis create a series of cashflows that would otherwise be achieved by a single financial instrument. Often the composite instruments are the underlying and its derivative. (Why not just buy the underlying? It may be difficult or more expensive to do so.)

For example, a synthetic call option can be created by buying the underlying as well as a put option. When prices goes up, you win. When price goes down, you don't lose money (apart from the premium you pay to acquire the put option). See this Investopedia [explainer](https://www.investopedia.com/articles/optioninvestor/08/synthetic-options.asp).

**(2) By structuring the ideal cashflows and finding a counterparty**

In this approach, an investor would create a bespoke series of cash flows, and find a counterparty who is willing to take the opposite side of the trade. This is often managed by a bank or similar that can create  complex financial structures and identify suitable counterparties.

For example, a credit default swap ("CDS") is a contract between two parties, a credit protection buyer and a credit protection seller. The buyer makes a series of cash payments to the seller and receives a promise of compensation for losses resulting from a “credit event”, such as a failure to pay, bankruptcy, or restructuring. This gives the CDS seller the ability to synthetically long an underlying asset and the CDS buyer the ability to hedge their exposure to an underlying asset. In exchange for taking on the risk of financial losses, the credit protection seller is paid the series of cash flows. (1kx has a [great explainer](https://medium.com/bollinger-investment-group/synthetic-assets-in-defi-use-cases-opportunities-19b11f57a776) on synthetics in DeFi from 2019.)

**Which model does Synthetix follow?**

A trader only needs to own the sBTC Synth in order to get full exposure to the price of bitcoin. This single-instrument set up means that Synthetix is unlikely to be using Model (1), which uses multiple instruments.

By elimination, and by intuition, it feels more like Synthetix is using Model (2), the crux of which are counterparties that take the opposite side of the bitcoin price exposure.

**So when someone buys sBTC, who is their counterparty?** Online explanations by Synthetix and third parties make direct and indirect claims that traders "do not need to worry about finding counterparties." This sounds too good to be true, which piqued my interest.

**Also, what what is the role of the SNX token?**
Are the economics of the system enough to sustain itself, without the SNX token incentives? Currently the sources of income or gains for stakers include:
* 0.3% fees charged by Kwenta, the platform that facilitates trading of Synths among each other
* rewards for staking
* SNX rising in price

____

### The Synthetix Staker's financial model

I spent a fair amount of time building this model. The design of Synthetix is deceptively simple, and I had to trawl a lot of explainers before I found ones that provided good and detailed enough explanations.

This model allows you to stake a variable amount of SNX, mint sUSD, then exchange some variable amount of sUSD to sBTC. You can then observe the effect it has on the staker's portfolio and debt liability.

The model also allows you to simulate the scenario where bitcoin changes in price adjust the value of Synths (sBTC in particular), to observe the effect it has on the Staker's P&L.

* Original model in Excel: [Excel model](~/2022-01-28_-_Synthetix_model.xlsx)

* [Google sheet version](https://docs.google.com/spreadsheets/d/1dSg_0sV7kBgdR911jB5-W6vDlghKe1MPSNO6Avm26ZI/). This was a simple Excel-Gsheets conversion, provided for convenience, and has not been checked for discrepancies that sometimes happen in conversions.

____

### Thoughts

Overall, the modeling illustrates that:

* Being an SNX staker without mirroring the Synth pool closely means that I am effectively short all the traders that participate in Synth trading.

It's not clear to me that this is a wise bet. You are accepting to be the counterparty of anyone who wants to buy a Synth. If he is a good trader, then you're going to be eating losses. It is counterintuitive to imagine that the way for Synthetix stakers to succeed is for them to attract bad traders rather than good traders.

* The model doesn't work without rich incentives programs.

Here are some reasons that maintaining a Synthetix staking position has significant barriers:
- The SNX:sUSD stake:mint ratio is 600% (aka the collateralization ratio). If i want to mint $1000 of sUSD, i need to buy and deposit $6000 worth of SNX. This feels capital inefficient
- Maintaining the collateralization ratio of 600% can be troublesome and expensive, as the value of the Synths and of SNX can change rapidly
At one point early on, the ratio of fees distributed to stakers from the Kwenta exchange to the value of SNX staking rewrds was about 1:10.

Let me know if you have feedback, further questions, or suggestions. DMs are open at Twitter @88crypt.
