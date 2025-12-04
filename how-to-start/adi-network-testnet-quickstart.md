---
description: Getting starting developing with ADI Network Testnet
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

#### 1. The Counter contract code

```solidity
contract Counter {
    uint256 public number;

    function setNumber(uint256 newNumber) public {
        number = newNumber;
    }

    function increment() public {
        number++;
    }
}
```

#### 2. Create a new Foundry project

```bash
forge init Counter
cd Counter
```

#### 3. Build the project

```bash
forge build
```

#### 4. Set your private key for deploying

```bash
export TESTNET_PRIVATE_KEY="0x..."
```

#### 5. Create `counter.sol`  file and put the contract code inside.

#### 6. Create `counter.s.sol` and put the script inside, the script will deploy the counter contract

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Script} from "forge-std/Script.sol";
import {Counter} from "../src/Counter.sol";

contract CounterScript is Script {
    Counter public counter;

    function setUp() public {}

    function run() public {
        vm.startBroadcast();

        counter = new Counter();

        vm.stopBroadcast();
    }
}
```

#### 7. Deploy the contract

```bash
forge script script/Counter.s.sol \
--rpc-url https://zksync-os-testnet-alpha.zksync.dev \
--broadcast --private-key $TESTNET_PRIVATE_KEY
```

#### 8. Set the number value

```bash
cast send 0xCA1386680bfd9D89c7cc6Fc3ba11938ba6E44fef \
"setNumber(uint256)" 5 \
--rpc-url https://rpc.ab.testnet.adifoundation.ai \
--private-key $TESTNET_PRIVATE_KEY
```

#### 9. Get the latest number value

```bash
cast call 0xCA1386680bfd9D89c7cc6Fc3ba11938ba6E44fef \
"number()" \
--rpc-url https://rpc.ab.testnet.adifoundation.ai
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
    adiTestnet: {
      type: 'http',
      chainType: 'generic',
      url: 'https://rpc.ab.testnet.adifoundation.ai',
      accounts: [configVariable('TESTNET_PRIVATE_KEY')],
    },
  },
```

#### 4. Ensure that the module file `ignition/modules/counter.ts` contains the following code&#x20;

```typescript
import { buildModule } from '@nomicfoundation/hardhat-ignition/modules';

export default buildModule('CounterModule', (m) => {
  const counter = m.contract('Counter');

  m.call(counter, 'increment', []);

  return { counter };
});
```

#### 5. Add your private key to the keystore as `TESTNET_PRIVATE_KEY` .

```bash
npx hardhat keystore set TESTNET_PRIVATE_KEY
```

#### 6. Compile and deploy the example contract

```bash
npx hardhat compile
npx hardhat ignition deploy ignition/modules/Counter.ts --network adiTestnet
```

#### 7. Create a new script file in the `scripts` folder called `increment.ts` .

```bash
touch scripts/increment.ts
```

#### 8. Copy/paste the script below.

```typescript
import { network } from 'hardhat';
import { type Abi, defineChain } from 'viem';

const CONTRACT_ADDRESS = 'THE_ADDRESS_OF_FRESHLY_DEPLOYED_CONTRACT';

const adiChain = defineChain({
  id: 36900,
  name: 'ADI Chain',
  network: 'adiTestnet',
  nativeCurrency: { name: 'ADI', symbol: 'ADI', decimals: 18 },
  rpcUrls: { default: { http: ['https://rpc.ab.testnet.adifoundation.ai'] } },
});

const { viem } = await network.connect('adiChain');

const publicClient = await viem.getPublicClient({ chain: adiChain });
const [senderClient] = await viem.getWalletClients({ chain: adiChain });
if (!senderClient) throw new Error('No wallet client. Set TESTNET_PRIVATE_KEY in hardhat config.');

const counterContract = await viem.getContractAt('Counter', CONTRACT_ADDRESS, {
  client: { public: publicClient, wallet: senderClient },
});

const initialCount = await publicClient.readContract({
  address: CONTRACT_ADDRESS,
  abi: counterContract.abi as Abi,
  functionName: 'number',
});
console.log('Initial count:', initialCount);

const tx = await senderClient.writeContract({
  address: CONTRACT_ADDRESS,
  abi: counterContract.abi as Abi,
  functionName: 'increment',
});
await publicClient.waitForTransactionReceipt({ hash: tx });
console.log('Transaction sent successfully');

const newCount = await publicClient.readContract({
  address: CONTRACT_ADDRESS,
  abi: counterContract.abi as Abi,
  functionName: 'number',
});
console.log('New count:', newCount);
```

#### 9. Run the script

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
    adiTestnet: {
      type: 'http',
      chainType: 'generic',
      url: 'https://rpc.ab.testnet.adifoundation.ai/',
      accounts: [configVariable('TESTNET_PRIVATE_KEY')],
    },
  },
```

#### 4. Add your private key to the keystore as `TESTNET_PRIVATE_KEY` .

```bash
npx hardhat keystore set TESTNET_PRIVATE_KEY
```

#### 5. Ensure that the module file `ignition/modules/counter.ts` contains the following code&#x20;

```typescript
import { buildModule } from '@nomicfoundation/hardhat-ignition/modules';

export default buildModule('CounterModule', (m) => {
  const counter = m.contract('Counter');

  m.call(counter, 'increment', []);

  return { counter };
});
```

#### 6. Compile and deploy the example contract.

```ts
npx hardhat compile
npx hardhat ignition deploy ignition/modules/Counter.ts --network adiTestnet
```

#### 7. Create a new script file in the `scripts` folder called `increment.ts`.

```bash
touch scripts/increment.ts
```

#### 8. Copy/paste the script below.

```typescript
import { network } from 'hardhat';

const CONTRACT_ADDRESS = 'THE_ADDRESS_OF_FRESHLY_DEPLOYED_CONTRACT';

const { ethers } = await network.connect({
  network: 'adiTestnet',
  chainType: 'generic',
});

const [sender] = await ethers.getSigners();

const contract = await ethers.getContractAt('Counter', CONTRACT_ADDRESS, sender);

const initialCount = await contract.x();
console.log('Initial count:', initialCount);

const tx = await contract.increment();
await tx.wait();
console.log('Transaction sent successfully');

const newCount = await contract.number();
console.log('New count:', newCount);
```

#### 9. Run the script

```bash
npx hardhat run scripts/increment.ts
```
{% endtab %}
{% endtabs %}

***
