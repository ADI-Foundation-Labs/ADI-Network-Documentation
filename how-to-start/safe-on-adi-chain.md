# Safe on ADI Chain

ADI Chain supports **Safe** (formerly Gnosis Safe), the leading smart account infrastructure for secure multi-signature and account abstraction setups.

#### Official Safe Interfaces & Services

* **Safe UI (Web Interface)**: [https://safe.adifoundation.ai/](https://safe.adifoundation.ai/)\
  Use this to create, manage, and interact with Safe accounts on ADI Chain (Mainnet and Testnet).
* **Mainnet Transaction Service**: [https://transaction.safe.adifoundation.ai](https://transaction.safe.adifoundation.ai)\
  Handles off-chain transaction proposing, signing, and indexing for Mainnet.
* **Testnet Transaction Service**: [https://transaction-testnet.safe.adifoundation.ai](https://transaction-testnet.safe.adifoundation.ai)\
  Same as above, but for the ADI Chain Testnet.

#### How to Use

1. Go to the [Safe UI](https://safe.adifoundation.ai/) and connect your wallet or create a new Safe.
2. The interface automatically uses the correct Transaction Service for ADI Chain.
3. For developers integrating Safe SDKs:
   * Use the Transaction Service URLs above when initializing `SafeApiKit` with a custom `txServiceUrl`.

Example with Safe Core SDK (TypeScript):

```ts
import SafeApiKit from '@safe-global/api-kit';

const apiKit = new SafeApiKit({
  chainId: <ADI_CHAIN_ID>,           // your ADI mainnet or testnet chainId
  txServiceUrl: 'https://transaction.safe.adifoundation.ai'  // or the testnet URL
});
```

{% hint style="info" %}
Always verify you are on the official domains listed above to avoid phishing risks.
{% endhint %}
