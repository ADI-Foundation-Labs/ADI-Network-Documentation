---

description: Deploy and manage L3 chains using the ADI CLI

---

# CLI

The ADI CLI is a Rust-based tool that manages the full lifecycle of L3 chain deployment. It runs all operations inside pre-built Docker toolkit containers (zkstack, foundry-zksync, era-contracts) and outputs the resulting state files and generated wallets to your host machine.

**Source**: [ADI-Foundation-Labs/ADI-CLI](https://github.com/ADI-Foundation-Labs/ADI-CLI)

## Prerequisites


| Requirement   | Details                                                                                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Docker        | Running daemon. The CLI pulls and runs toolkit images automatically                                                                                                 |
| Rust          | Install via [rustup](https://rustup.rs/)                                                                                                                            |
| Funded wallet | ~270 ADI tokens on the settlement layer for deployment gas costs. Testnet faucet: [faucet.ab.testnet.adifoundation.ai](https://faucet.ab.testnet.adifoundation.ai/) |


## Installation

```bash
cargo install --git https://github.com/ADI-Foundation-Labs/ADI-CLI
```

Verify the installation:

```bash
adi version
```

Optionally generate shell completions:

{% tabs %}
{% tab title="Zsh" %}
```bash
mkdir -p ~/.oh-my-zsh/completions
adi completions zsh > ~/.oh-my-zsh/completions/_adi
```
{% endtab %}

{% tab title="Bash" %}
```bash
mkdir -p ~/.local/share/bash-completion/completions/
adi completions bash > ~/.local/share/bash-completion/completions/adi
```
{% endtab %}
{% endtabs %}

Restart your shell to activate:

{% tabs %}
{% tab title="Zsh" %}
```bash
source ~/.zshrc
```
{% endtab %}

{% tab title="Bash" %}
```bash
source ~/.bashrc
```
{% endtab %}
{% endtabs %}

## Configuration

The CLI reads configuration from a YAML file. Default location is `~/.adi.yml`.

{% hint style="info" %}
Override the config path with the `--config` flag or `ADI_CONFIG` environment variable.
{% endhint %}

### Minimal Config

```yaml
protocol_version: v0.30.1

ecosystem:
  name: my-ecosystem
  rpc_url: https://rpc.ab.testnet.adifoundation.ai
  chains:
    - name: my-chain
      chain_id: 222
      prover_mode: gpu   # no-proofs (testing) | gpu (production)
      base_token_address: "0x..." # custom gas token address, or omit for ETH
```

{% hint style="info" %}
`protocol_version` determines which Docker toolkit image the CLI uses. Each protocol version is tied to a specific server version (e.g., protocol `v0.30.1` works with server `v13.1`).
{% endhint %}

Wallet funding amounts and the funder key are not required in the config — they can be provided via CLI flags or environment variables during `adi deploy`. The only required config fields are the ecosystem definition and chain parameters.

### Key Options


| Option                       | Scope           | Description                                      |
| ---------------------------- | --------------- | ------------------------------------------------ |
| `base_token_address`         | chain           | Custom Gas Token (CGT) contract address on L2    |
| `governor_cgt_units`         | funding         | Amount of CGT to fund the governor wallet        |
| `operators.operator`         | chain/global    | Address for batch commit/revert roles            |
| `operators.prove_operator`   | chain/global    | Address for proof submission                     |
| `operators.execute_operator` | chain/global    | Address for batch execution                      |
| `ownership.new_owner`        | ecosystem/chain | Transfer ownership to this address after deploy  |
| `ownership.private_key`      | ecosystem/chain | If provided, ownership is accepted automatically |
| `gas_multiplier`             | global          | Gas price buffer percentage (default: 200)       |


Set the funder wallet private key via environment variable:

```bash
export ADI_FUNDER_KEY="0x..."
```

{% hint style="warning" %}
Never put private keys directly in the config file. Use `ADI_FUNDER_KEY` for the funder and `ownership.private_key` only when automatic acceptance is needed.
{% endhint %}

For the full annotated config, environment variables, and S3 state sync, see the [CLI Configuration Reference](../../references/cli-configuration.md).

### Ownership Transfer

{% hint style="warning" %}
When `ownership.new_owner` is set:

* **Address only** — ownership is transferred after deployment. The new owner must run `adi accept` to complete the transfer.
* **Address + `private_key`** — ownership is transferred and accepted automatically during deployment.
{% endhint %}

Verify the merged configuration:

```bash
adi config
```

## Deployment Workflow

### 1. Initialize

The `init` command creates the ecosystem and chain configuration. It spins up a Docker container, generates wallets and configs inside it, then drops the resulting state files to your host.

```bash
adi init --chain my-chain
```


| Flag      | Description                                 |
| --------- | ------------------------------------------- |
| `--chain` | Select chain from config `chains[]` by name |
| `--force` | Overwrite existing ecosystem state          |
| `--yes`   | Skip confirmation prompts                   |

{% hint style="info" %}
If you have a single ecosystem and chain configured, the CLI selects them automatically. Otherwise it will prompt you to choose. You can also specify `--chain` explicitly.
{% endhint %}

Example output:

```
┌   ADI Init
│
✱  Select a chain
│  my-chain
│
◈  Validating chain ID against settlement layer...
│
◆  Chain ID 222 validated (settlement layer: 99999)
│
✱  Protocol version: v0.30.1 ──────╮
│                                  │
│  Ecosystem: adi_ecosystem        │
│  Chain: my-chain (ID: 222)       │
│  Prover mode: gpu                │
├──────────────────────────────────╯
│
◈  Connecting to Docker...
│
◈  Running zkstack ecosystem create...
│
◆  Completed in 0s
│
◈  Importing ecosystem state...
│
◆  Ecosystem state imported successfully
│
◈  Validating imported state...
│
◆  State validated: 1 chain(s) found
│
◈  Location: ~/.adi_cli/state/adi_ecosystem
│
└   Ecosystem 'adi_ecosystem' initialized successfully!
```

After init, state is written to `~/.adi_cli/state/<ecosystem>/`:

```
~/.adi_cli/state/my-ecosystem/
├── ZkStack.yaml
├── configs/
│   ├── wallets.yaml        # generated ecosystem wallets
│   ├── contracts.yaml
│   └── ...
└── chains/my-chain/
    ├── ZkStack.yaml
    └── configs/
        ├── wallets.yaml    # generated chain wallets (operators)
        ├── genesis.yaml
        └── ...
```

### 2. Deploy

The `deploy` command funds the generated wallets and deploys all contracts to the settlement layer (L2).

The funder private key can be provided in three ways (in priority order): `--funder-key` flag, `ADI_FUNDER_KEY` env var, or `funding.funder_key` in config.

```bash
export ADI_FUNDER_KEY="0x..."
adi deploy
```

Example output:

```
┌   ADI Deploy
│
✱  Deployment target ───────────────────────────────────────────────╮
│                                                                   │
│  Ecosystem: adi_ecosystem                                         │
│  Chain: my-chain (L3)                                             │
│  Settlement layer RPC: https://rpc.ab.testnet.adifoundation.ai/   │
├───────────────────────────────────────────────────────────────────╯
│
...
│
✱  Funding Summary ──────────────────────╮
│                                        │
│  Transfers needed: 6                   │
│  Total ETH to transfer: 270.0000 ETH   │
│  Estimated gas cost: 0.1401 ETH        │
│  Total ETH required: 270.1401 ETH      │
│  Funder balance: 1979.2569 ETH         │
│  Status: Sufficient balance            │
├────────────────────────────────────────╯
│
...
│
✱  Deployed Contracts ──────────────────────────────────────────────╮
│                                                                   │
│  Diamond proxy: 0x5b0323f95682228DE5b0db338bA9470D2D511F80        │
│  Validator timelock: 0x686B1825b998007825ad4A34E0c00fF361D0667c   │
│  Chain admin: 0x195Ec0826b0686D03E21De802588a28f253F69C7          │
├───────────────────────────────────────────────────────────────────╯
│
✱  Deployment Summary ─────────────────────────────────────────╮
│                                                              │
│  Ecosystem: adi_ecosystem                                    │
│  Chain: my-chain                                             │
│  Diamond proxy: 0x5b0323f95682228DE5b0db338bA9470D2D511F80   │
├──────────────────────────────────────────────────────────────╯
│
└   Deployment complete! You can now start containers and operate the rollup.
```

This command:

1. Connects to the settlement layer and checks wallet balances
2. Funds deployer, governor, and operator wallets from the funder
3. Deploys ecosystem and chain contracts (Bridgehub, STM, Diamond Proxy)
4. Configures validator roles on chain contracts
5. Handles ownership transfer if configured

## State

All state lives in `~/.adi_cli/state/`. This directory contains wallets (with private keys), deployed contract addresses, and chain configurations. Back it up.

State can optionally sync to S3-compatible storage with `adi state sync` / `adi state restore`. See the [CLI Configuration Reference](../../references/cli-configuration.md#s3-state-synchronization) for details.

## Next Steps

Once contracts are deployed, set up the chain infrastructure (sequencer, prover, explorer, bridge) using Docker Compose:

{% content-ref url="run-a-rollup/" %}
# [Infrastructure Setup](run-a-rollup/)
Infrastructure Setup
{% endcontent-ref %}

Additional CLI commands for post-deployment:


| Command             | Purpose                                                   |
| ------------------- | --------------------------------------------------------- |
| `adi verify`        | Verify deployed contracts on block explorers              |
| `adi server-params` | Output server parameters for docker-compose configuration |
| `adi owners`        | Display current contract owners                           |
| `adi transfer`      | Transfer ownership to a new address                       |
| `adi upgrade`       | Upgrade contracts to a new protocol version               |


