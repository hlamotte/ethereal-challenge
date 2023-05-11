https://hyperdrive.delv.tech/

Hyperdrive is the next research leap from the Delv team on variable and fixed rate primitives. 
- No preset expiration dates
- no fragmented liquidity
- no LP rollovers, aka everlasting liquidity.

Rebalance holdings to avoid impermanent loss. Impermanent loss caused by you only receiving the fraction of the liquidity pool assets you deposited, so if the price ratio changes you get impermanent loss.

ETH =  1K USDC, deposit 1ETH & 1K USDC = 2K USDC
Future: ETH = 0.5K USDC, so = 1.5K USDC

LP rollover = need to leave and then rejoin the pool as market conditions have drifted

https://www.youtube.com/watch?v=aTp9er6S73M&ab_channel=Finematics

The CAPM formula is expressed as follows: Expected Return = Risk-free rate + Beta * (Market return - Risk-free rate)

Element Fi wraps existing yield protocol and creates fixed APY opportunities.

When you deposit to Element Fi wrapped protocol, you get PT and YT. PT can be resold as fixed APY asset, with APY determined by price someone willing to pay for opportunity cost.

# Modeling notes
Objective: MSI correctly tracks crypto equivalent of mean risk adjusted (or "risk-free") returns on the market? Eg - comparison to US Treasury Bond yields.

Actors in market should be correctly incentivised by AMM to interact with it such that objective is achieved.

Can I think of "HyperDrive" as a market for those PTs, wrapping other lending protocols?

Design:
Long FR -> drives MSI down (fixed-rate deposit?) (think price of PT is going to increase, and could sell for profit?) (buy shares)

Short FR -> drives MSI up (fixed-rate borrow) (think price of PT is going to decrease, and could sell for profit?)

Some factors that will influence it:
- APY on large market cap yield farming opportunities like Dai Savings Rate as arbitrage opportunities

Playing around with influence of market conditions: https://github.com/hlamotte/elf-simulations/blob/main/examples/notebooks/hamish_expts/frida_louie_simulation_copy.ipynb

- Hyperdrive.ipynb does not implement agents that respond to market conditions
- Adding more than 200 random agents modeling becomes very slow.
- Modeling down to many agents small agents doesn't scale well, can we model aggregate behaviour of many actors as a single agent?
- More efficient if we instantiate agent typologies (aggregate agents), rather than many individual agents?

What is the interface with the market:
```
- ProvideLiquidity(amount)
- OpenShort(amount) # need to be able to close positions and manage multiple positions
- OpenLong(amount)
- CurrentlyOpenPositions(): List[Positions]
```
- Side thought - are there ERC standards for the interface of DeFi lending protocols, or DeFi protocols in general?

- Could it be beneficial to de-couple the protocol implementation code from the protocol user-modeling code?
    - Seems to me agent typologies generally tend to span many different protocols.
    - Could leverage open-source community to create a repo for testing DeFi protocols with rich functionality which over-time could become a useful resource for battle testing protocols

Different agents they've tried modeling:
- FixedFrida
    - High risk threshold
    - Open short if fixed rate 2% higher than variable
    - Won't close until simulation stops
- LongLouie
    - Open long at a random point if my position will not cause the fixed rate APR go below variable rate
    - Won't close until position matures

FridaLouieSimulation:
- (Simplistic) Frida makes profit when at close time fixed rate is below when she opened short position.
- Louie profits when at close time fixed rate is above when he opened position.
- Long position / liquidity provisioning increases share reserves
- Short position increases bond reserves
- Default simulation there were 20x more Louies than Frida - capital available to execute longs dominated that of shorts
- In three different market conditions, with otherwise default setup MSI initially seems to target variable rate, but then becomes unstable and trends upwards. Shorts dominate longs in the long term.
- Longs are stop being made in second half, is that because positions are opened but never closed.
- Need to introduce other agents, or add "smartness" to these agents as system becomes unstable.

Other possible agents to consider:
- BorrowingBeatrice
    - (Non-speculator) Need to borrow money for a project, and will borrow when fixed rate is below a threshold.
- AbritrageAndy
    - If there is another market with a different rate I can use arbitrage to at one and lend on the other and profit on the discrepancy.

- Could we make use of some on-chain data of existing lending protocols (Aave, Compound, MakerDao, etc.) to develop some data-driven agent typologies?
    - How did their LPs and positions vary over time since adoption?
    - Will be coupled to "macro" conditions of crypto market
    - Are there other similar lending protocols that have recently launched that we could analyse (as under similar market "macro" conditions) and model how hyperdrive will respond to those market conditions and agent types
- Model a situation where there are multiple protocols with different variable lending rates, and how they interact with MSI

- Checking on-chain Aave has 200 DAU for a random day, but will want to model for challenging market conditions, would be interested to see how interactions with Aave change during rate change announcment etc.

- See Compound adoption stats here: https://dune.com/messari/Messari:-Compound-Macro-Financial-Statements

