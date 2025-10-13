---
description: Getting starting developing with ADI Network Testnet
hidden: true
---

# ADI Network Testnet Quickstart

You can use all standard EVM tooling for ADI Network Testnet. You can find instructions for bridging testnet ADI and ETH to the ADI Network Testnet in the [Network Details](adi-network-testnet-details.md) page.

To get started with Foundry or Hardhat, follow the steps below:

## Deploying to ADI Network Testnet

Below are setup examples for **Foundry** and **Hardhat 3** (with Viem or Ethers).\
Select your preferred setup tab:

***

{% tabs %}
{% tab title="Foundry Setup" %}
## Foundry Setup

#### 1. Create a new Foundry project

```bash
forge init Counter
cd Counter
```

#### 2. Build the project

```bash
forge build
```

#### 3. Set your private key for deploying

```bash
export TESTNET_PRIVATE_KEY="0x..."
```

#### 4. Deploy the contract

```bash
forge script script/Counter.s.sol \
--rpc-url https://zksync-os-testnet-alpha.zksync.dev \
--broadcast --private-key $TESTNET_PRIVATE_KEY
```

#### 5. Set the number value

```bash
cast send 0xCA1386680bfd9D89c7cc6Fc3ba11938ba6E44fef \
"setNumber(uint256)" 5 \
--rpc-url https://rpc.testnet.adifoundation.ai \
--private-key $TESTNET_PRIVATE_KEY
```

#### 6. Get the latest number value

```bash
cast call 0xCA1386680bfd9D89c7cc6Fc3ba11938ba6E44fef \
"number()" \
--rpc-url https://rpc.testnet.adifoundation.ai
```
{% endtab %}

{% tab title="Hardhat 3 with Viem" %}
## Hardhat 3 with Viem

#### 1. Create a new project folder

```bash
mkdir hardhat-example
cd hardhat-example
```

#### 2. Initialize a new Hardhat 3 project with Node Test Runner and Viem.

```bash
npx hardhat --init
```

#### 3. Add ADI Network Testnet to `hardhat.config.ts` file and configure the ignition required confirmations.

```ts
  ignition: {
    requiredConfirmations: 1,
  },
  networks: {
    ADITestnet: {
      type: 'http',
      chainType: 'generic',
      url: 'https://rpc.testnet.adifoundation.ai',
      accounts: [configVariable('TESTNET_PRIVATE_KEY')],
    },
  },
```

#### 4. Add your private key to the keystore as `TESTNET_PRIVATE_KEY` .

```bash
npx hardhat keystore set TESTNET_PRIVATE_KEY
```

#### 5. Compile and deploy the example contract

```bash
npx hardhat compile
npx hardhat ignition deploy ignition/modules/Counter.ts --network ADITestnet
```

#### 6. Create a new script file in the `scripts` folder called `increment.ts` .

```bash
touch scripts/increment.ts
```

#### 7. Copy/paste the script below.

```typescript
import { network } from 'hardhat';
import { type Abi, defineChain } from 'viem';

const CONTRACT_ADDRESS = '0x7Be3f2d08500Fe75B92b9561287a16962C697cb7';

const { viem } = await network.connect('zksyncOS');

const zksyncOS = defineChain({
  id: 8022833,
  name: 'ZKsync OS',
  network: 'zksyncOS',
  nativeCurrency: { name: 'Ether', symbol: 'ETH', decimals: 18 },
  rpcUrls: { default: { http: ['https://zksync-os-testnet-alpha.zksync.dev'] } },
});

const publicClient = await viem.getPublicClient({ chain: zksyncOS });
const [senderClient] = await viem.getWalletClients({ chain: zksyncOS });
if (!senderClient) throw new Error('No wallet client. Set TESTNET_PRIVATE_KEY in hardhat config.');

const counterContract = await viem.getContractAt('Counter', CONTRACT_ADDRESS, {
  client: { public: publicClient, wallet: senderClient },
});

const initialCount = await publicClient.readContract({
  address: CONTRACT_ADDRESS,
  abi: counterContract.abi as Abi,
  functionName: 'x',
});
console.log('Initial count:', initialCount);

const tx = await senderClient.writeContract({
  address: CONTRACT_ADDRESS,
  abi: counterContract.abi as Abi,
  functionName: 'inc',
});
await publicClient.waitForTransactionReceipt({ hash: tx });
console.log('Transaction sent successfully');

const newCount = await publicClient.readContract({
  address: CONTRACT_ADDRESS,
  abi: counterContract.abi as Abi,
  functionName: 'x',
});
console.log('New count:', newCount);
```

#### 8. Run the script

```bash
npx hardhat run scripts/increment.ts
```
{% endtab %}

{% tab title="Hardhat 3 with Ethers" %}
## Hardhat 3 with Ethers

#### 1. Create a new project folder

```bash
mkdir hardhat-example
cd hardhat-example
```

#### 2. Initialize a new Hardhat 3 project with Mocha and Ethers.js.

```bash
npx hardhat --init
```

#### 3. Add ADI Network Testnet to the `hardhat.config.ts` file and configure the ignition required confirmations.

```bash
  ignition: {
    requiredConfirmations: 1,
  },
  networks: {
    zksyncOS: {
      type: 'http',
      chainType: 'generic',
      url: 'https://zksync-os-testnet-alpha.zksync.dev',
      accounts: [configVariable('TESTNET_PRIVATE_KEY')],
    },
  },
```

#### 4. Add your private key to the keystore as `TESTNET_PRIVATE_KEY` .

```bash
npx hardhat keystore set TESTNET_PRIVATE_KEY
```

#### 5. Compile and deploy the example contract.

```ts
npx hardhat compile
npx hardhat ignition deploy ignition/modules/Counter.ts --network zksyncOS
```

#### 6. Create a new script file in the `scripts` folder called `increment.ts`.

```bash
touch scripts/increment.ts
```

#### 7. Copy/paste the script below.

```typescript
import { network } from 'hardhat';

const CONTRACT_ADDRESS = '0x8e882b31Fe1d3942c57408D354E754d1659400a7';

const { ethers } = await network.connect({
  network: 'zksyncOS',
  chainType: 'generic',
});

const [sender] = await ethers.getSigners();

const contract = await ethers.getContractAt('Counter', CONTRACT_ADDRESS, sender);

const initialCount = await contract.x();
console.log('Initial count:', initialCount);

const tx = await contract.inc();
await tx.wait();
console.log('Transaction sent successfully');

const newCount = await contract.x();
console.log('New count:', newCount);
```

#### 8. Run the script

```bash
npx hardhat run scripts/increment.ts
```
{% endtab %}
{% endtabs %}

***
