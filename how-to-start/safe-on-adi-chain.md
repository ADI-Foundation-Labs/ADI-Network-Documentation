---
description: Official Safe interfaces and transaction services for ADI Chain.
---

# Safe on ADI Chain

ADI Chain supports **Safe** (formerly Gnosis Safe), the smart account infrastructure for multi-signature accounts and account abstraction.

#### Official Safe interfaces and services

* **Safe UI (web interface)**: [https://safe.adifoundation.ai/](https://safe.adifoundation.ai/)\
  Redirects to the Protofire-hosted Safe deployment for ADI. Use it to create, manage, and interact with Safe accounts on ADI Mainnet (chain ID `36900`) and ADI Network AB Testnet (chain ID `99999`).
* **Mainnet transaction service**: `https://transaction-adi.safe.protofire.io/api`\
  Off-chain proposing, signing, and indexing for mainnet. `GET /api/v1/about/` reports `Safe Transaction Service 5.43.0`.
* **Testnet transaction service**: `https://transaction-adi-testnet.safe.protofire.io/api`\
  Same service for the testnet.

{% hint style="warning" %}
Use the transaction service URLs as written, including the `/api` suffix. The host root returns 403, and paths without `/api` do not resolve. The previous `transaction.safe.adifoundation.ai` and `transaction-testnet.safe.adifoundation.ai` hostnames do not resolve in DNS.
{% endhint %}

#### How to use

1. Open the [Safe UI](https://safe.adifoundation.ai/) and connect your wallet or create a new Safe.
2. The interface selects the correct transaction service for the selected chain.
3. For Safe SDK integrations, pass the full service URL, `/api` included, as `txServiceUrl`.

Example with Safe Core SDK (TypeScript), verified against both services:

```ts
import SafeApiKit from '@safe-global/api-kit';

const apiKit = new SafeApiKit({
  chainId: 36900n,                                          // 99999n for testnet
  txServiceUrl: 'https://transaction-adi.safe.protofire.io/api',
});

const info = await apiKit.getServiceInfo();
// { name: 'Safe Transaction Service', version: '5.43.0', api_version: 'v1', ... }
```

{% hint style="info" %}
Always verify you are on the official domains listed above to avoid phishing risks. `safe.adifoundation.ai` is the only first-party Safe UI; the transaction services are operated on `safe.protofire.io`.
{% endhint %}
