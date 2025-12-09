# What is MEV? How Can You Avoid It?

When you use the blockchain, you might hear the term **MEV** — it stands for _Maximal Extractable Value_. In simple terms: MEV is the additional value that can be captured by a block-producer (or someone who can influence block ordering) beyond the standard rewards and transaction fees.

### Why MEV matters

* On a blockchain like ADI or Ethereum, every transaction goes into a pool (mempool, where pending transactions are placed) where it waits before being included in a block.
* Whoever builds the next block has some discretion: they can **include**, **exclude**, or **re-order** transactions.
* That discretion creates an opportunity: by changing the order or by inserting their own transactions, someone can capture extra profit.
* For you, MEV means your transaction outcome could be affected (for example, worse price, higher slippage).

### What MEV Means for You

* Your transaction might **face higher cost** (more slippage, worse execution) because of bots or searchers extracting value ahead of you.
* You might **experience delay or surprise ordering**: your transaction may not execute in the order you expected.

## How Can You Avoid MEV?

### ADI Keeps Your Transactions Private

ADI uses a private pool. This means your pending transaction is not visible to public searchers. This blocks classic mempool sniping and most sandwich setups. It does not prevent economic MEV that comes from protocol rules, liquidity design, or block construction choices.

#### What You Should Do on Ethereum or Other Chains

ADI's private mempool protection works only when your transaction is executed on ADI. If you bridge assets to Ethereum or use other chains, your transactions enter public mempools or non private L2 sequencing pipelines. We recommend using the ADI wallet (to be released soon) for ensuring you are protected when interacting with the wider Ethereum ecosystem.

**ADI Wallet (Coming Soon)**

The ADI wallet is the official wallet to use with the ADI ecosystem of projects. It provides:

* **Within ADI**: native staking for ADI and the Dirham stablecoin
* **Broader Ethereum Ecosystem**: built-in MEV protection through OneInch transaction routing for swaps included in the wallet

### Further Reading

#### How MEV works

1. A user submits a transaction → it lands in the mempool.
2. A “searcher” (algorithm or bot) watches the mempool and identifies profitable opportunities (e.g., two DEXes have a price mismatch).
3. The searcher builds a _bundle_ (set or group) of transactions or chooses a transaction ordering. They may pay higher fees to ensure inclusion.
4. The block-builder (and ultimately the validator) selects which transactions (and in what order) to include.
5. Value is extracted beyond normal fees: e.g., via arbitrage, sandwiching, or liquidation trades.
   * This extra profit is what we call MEV.
   * Some of that profit is passed to the validator as higher gas/tip; some is captured by searchers.

#### Common MEV strategies

* **Arbitrage**: Exploit a price difference between two markets or liquidity pools. Example: Buy token X on DEX A at a lower price, sell on DEX B at higher price, all within one block.
* **Sandwich attacks**: A searcher sees your large trade pending. They place a buy _before_ your trade (paying high fee) to push the price up, then your trade executes at worse price, then they sell right after you — capturing profit at your expense.
* **Front-running / Back-running**: In front-running you get someone else's transaction’s effect ahead of them; in back-running you follow it to capture the after-effect.
* **Liquidation capture**: On lending protocols, when someone’s collateral becomes under-collateralised, an attacker can jump in, liquidate and profit.

**Further Resources**

* [A16zCrypto’s “MEV explained”](https://a16zcrypto.com/posts/article/mev-explained/)
* [Chainlink Education Hub’s “What is MEV?” deep dive](https://chain.link/education-hub/maximal-extractable-value-mev)
* [Arkham 2025 “MEV: A 2025 guide to Maximal Extractable Value in crypto”](https://info.arkm.com/research/beginners-guide-to-mev)
