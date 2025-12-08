# Gas Abstraction & Account Abstraction

You can leverage either native EVM flows or overlay ERC-4337-style account abstraction.

We do not ship audited “default paymaster” contracts. You are free to integrate with external AA infrastructure or deploy and own your own paymaster / smart-account stack.

{% hint style="warning" %}
What is not supported currently:

* ERC-7702
{% endhint %}

### Supported AA Paths

For both paths, we recommend auditing as a final step.

#### ERC-4337 via Pimlico & `permissionless.js`

Using [Pimlico](https://docs.pimlico.io/) + `permissionless.js` gives you a fully functional smart-account and paymaster stack on any EVM-compatible network, including zkOS.

We recommend using the battle-tested [Pimlico Bundler](https://docs.pimlico.io/references/bundler).&#x20;

{% hint style="info" %}
**Note**: at time of writing, the Bundler does not work in safe mode due to missing `debug_trace` support on the RPC, therefore other bundlers will likely also not work if they require safe mode by default.
{% endhint %}

Smart account that works flawlessly on ADI Chain (`v3.1` is used for the official non-custodial ADI Wallet), MIT licensed: [https://github.com/zerodevapp/kernel](https://github.com/zerodevapp/kernel)

#### Custom Paymaster / Smart-Account Deployment

You may deploy your own paymaster contracts and smart accounts. If you choose this path:

* Use a battle-tested base (e.g. a known ERC-4337 Paymaster template)
* Adapt to your tokenomics, gas-sponsorship rules, UI requirements

### Contracts

Entrypoints `V0.7` and `V0.8` are deployed on the network:

* `V0.7`: `0x0000000071727De22E5E9d8BAf0edAc6f37da032`
* `V0.8`: `0x4337084d9e255ff0702461cf8895ce9e3b5ff108`

### Further Reading

### Series of Articles from Alchemy on AA

* [https://www.alchemy.com/blog/account-abstraction](https://www.alchemy.com/blog/account-abstraction)
* [https://www.alchemy.com/blog/account-abstraction-paymasters](https://www.alchemy.com/blog/account-abstraction-paymasters)
* [https://www.alchemy.com/blog/account-abstraction-wallet-creation](https://www.alchemy.com/blog/account-abstraction-wallet-creation)
* [https://www.alchemy.com/blog/account-abstraction-aggregate-signatures](https://www.alchemy.com/blog/account-abstraction-aggregate-signatures)

#### More Solidity-Focused Docs from OpenZeppelin

* On smart accounts: [https://docs.openzeppelin.com/community-contracts/account-modules](https://docs.openzeppelin.com/community-contracts/account-modules)
* On paymasters: [https://docs.openzeppelin.com/community-contracts/paymasters](https://docs.openzeppelin.com/community-contracts/paymasters)

#### Paymaster Examples

* Pimlico: [https://github.com/pimlicolabs/singleton-paymaster](https://github.com/pimlicolabs/singleton-paymaster)
* Baseline example from `eth-infinitism` : [https://github.com/eth-infinitism/account-abstraction/blob/develop/contracts/core/BasePaymaster.sol](https://github.com/eth-infinitism/account-abstraction/blob/develop/contracts/core/BasePaymaster.sol)
