# Bridge activity dashboard

Existing bridge total TVL query: https://dune.com/queries/185131/345531

DeFi Llama bridges dashboard great starting point: https://defillama.com/bridges

- DeFi Llama data is biased towards whatever bridges they decide to list, but we can assume that they do a pretty good job as their listings are open source anyone can contribute a new project.
    - Could scrape bridge contracts from DeFi Llama source code or just manually on UI to begin with.
    - Wonder where project categorisation is defined on DeFi Llama

- If we wanted we could try and develop a methodolgy for discovering newly emerging bridges that were gaining popularity before they have large TVL. But not going to try and do "bridge discovery" using on-chain data

- Looking through the different data models and abstractions that Dune has:
    - Raw blockchain data is separated by chain, has as different data model for each chain, and effectively has a different data model for each contract making it difficult to get a birds eye view on the ecosystem.
    - Their decoded projects view works well for analysis of a specific bridge on a specific network, though seems a few of the bridges have not been successfully decoded/ingested as missing data, but again not great for birds eye view.
    - Approach for spells for more unified data models seems most useful, and the ERC20 table in particular if we make the assumption that most important bridges work with ERC20s both ends

Build a chain agnostic ERC20 dataset for transfer events with top3 chains by "native token" market cap (from CoinMarketCap):
1. Ethereum
1. BSC
1. Polygon


Needs to be a blockchain supported by the Dune ERC20 spell - only going to cover EVM chains.

Useful spell tables:
1. `prices.usd` - for finding number of decimals in a ERC20 token (not needed)
1. 

Create dashboard with number of weekly transactions for a handful of bridges

Getting data into USD pricing a lot of work in Dune, I'd create a Spell that's already done it.

Trouble with Spells is need to dig into the code of the pipeline of each one to understand the quality of the data and the use case it was intended for.

Simplify problem by just looking at Ethereum side of bridges and investigate number of weekly transactions (most bridges have Ethereum on one side)

Start with WBTC and MULTI as contract infrastructure straightforward to understand on ethereum:
- WBTC = `0x2260fac5e5542a773aa44fbcfedf7c193bc2c599` is an ERC20
- Multichain = `0x9Fb9a33956351cf4fa040f65A13b835A3C8764E3` is not an ERC20
- Wormhole/Portal = `0x3ee18B2214AFF97000D974cf647E7C347E8fa585` is not an ERC20
- RenBTC = `0xEB4C2781e4ebA804CE9a9803C67d0893436bB27D` is an ERC20

Example of sophisticated bridge TVL query handles each bridge data structure individually: https://dune.com/queries/118245/284806


- When each protocol has a different implementation/interface makes difficult to handle when want to get a birds eye view of many protocols
- Logs table is pretty valuable when it comes to analysing interactions with contracts


For total USD value locked and transfered calculations a USD transaction graph view of the ecosystem would be really helpful on Dune.

Dashboard I made: https://dune.com/hlamotte96/ethereal-demo

- No project categorisation/labelling on Dune makes it difficult to find new projects, maybe could enrich from another data source, like scraping GitHub.

- TVL calculation is more work as need to join with pricing data also and getting entire project TVL can span multiple contracts and difficult to link them together with on-chain data

- Handling projects with different contract interfaces is a real challenge, in an ideal world project developers would be incentivised to conform to standards or publish tooling to convert their new interface into standard data structures like those in Dune.

- I'd re-design Dune to have data models decoupled from application implementation and network, ie chain and application agnostic, but rather a data schema oriented around the types of questions that a user wants to answer such as.

In most cases users care about how money is moving around the blockchain ecosystem. I'd have a data model where all interactions were cast into "value transfer events", with a schema along the lines of:

- value transfer datetime
- value transfer event type (struct)
    - network
    - asset
    - block hash
    - transaction hash
- sender (struct)
    - project name
    - project category
    - address
- recipient (struct)
    - project name
    - project category
    - address
- native value
- usd value
- value transfer metadata (defined by event type) (struct)
    - log index
    - relevant function calls
    - state changes

This data model could be used to calculate which projects are seeing high interaction value volumes and the typologies of these interactions.

Incentivise projects to provide a mapping for their protocol to this schema so that there data will be available.

Just having the SQL interface for Dune, it's hard to develop your own data models for your own applications, and it's not very human readable. As analytics/BI people tend to be most familiar with SQL I understand why they went with this approach, but it would be helpful to have your own intermediary tables in this case.

Software stack for Dune:
- Spark data pipelines (Databricks)
- S3 for storage of delta tables