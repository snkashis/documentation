---
layout: ../../layouts/MainLayout.astro
section: ethereum
date: Last Modified
title: "EthBalanceMonitor Contract"
whatsnext: { "FAQs": "/chainlink-automation/faqs/" }
setup: |
  import CodeSample from "@components/CodeSample/CodeSample.astro"
---

This guide explains the use case for the [`EthBalanceMonitor` contract](https://github.com/smartcontractkit/chainlink/blob/master/contracts/src/v0.8/upkeeps/EthBalanceMonitor.sol). This Automation contract monitors and funds Ethereum addresses that developers might need to top up frequently based on a configurable threshold. As a result, nodes are funded automatically.

After deploying the contract, developers can go to [automation.chain.link](https://automation.chain.link/) to register Upkeep and run the contract. To take full advantage of the Chainlink Automation infrastructure, read all of the documentation to understand the features of Chainlink Automation.

To find other example contracts, see the [Example Automation Contracts](/chainlink-automation/util-overview/) page.

## `EthBalanceMonitor` Properties

`EthBalanceMonitor` is ownable, pauseable, and compatible with the `AutomationCompatibleInterface` contract:

- **Ownable**: The contract has an owner address, and provides basic authorization control functions. This simplifies the implementation of _user permissions_ and allows for transfer of ownership.
- **Pauseable**: This feature allows the contract to implement a pause and unpause mechanism that the contract owner can trigger.
- **Compatible**: The `AutomationCompatibleInterface` is necessary to create contracts that are compatible with the Chainlink Automation Network. To learn more about the `AutomationCompatibleInterface` and its uses and functions, read the [Making Compatible Contracts](/chainlink-automation/compatible-contracts/) guide.

You can open the contract in Remix:

<!-- prettier-ignore -->
<CodeSample src="samples/Automation/EthBalanceMonitor.sol" showButtonOnly/>

:::note[Note on Owner Settings]
Aside from certain features listed below, only owners can withdraw funds and pause or unpause the contract. If the contract is paused or unpaused, it will affect `checkUpkeep`, `performUpkeep`, and `topUp` functions.
:::

## Functions

Functions with an asterisk (`*`) denote features that only the owner can change. Click on each function to learn more about its parameters and design patterns:

| Function Name                                                      | Description                                                                            |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| [`setWatchList`\*](#setwatchlist-function)                         | Addresses to watch minimum balance and how much to top it up.                          |
| [`setKeeperRegistryAddress`\*](#setkeeperregistryaddress-function) | Updates the `KeeperRegistry` address.                                                  |
| [`setMinWaitPeriodSeconds`\*](#setminwaitperiodseconds-function)   | Updates the global minimum period between top ups.                                     |
| [`topUp`](#topup-function)                                         | Used by `performUpkeep`. This function will only trigger top up if conditions are met. |

Below are the feed functions in `EthBalanceMonitor`:

| Read Function Name         | Description                                                                                                                         |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `getUnderfundedAddresses`  | View function used in `checkUpkeep` to find underfunded balances.                                                                   |
| `getKeeperRegistryAddress` | Views the `KeeperRegistry` address.                                                                                                 |
| `getMinWaitPeriodSeconds`  | Views the global minimum period between top ups.                                                                                    |
| `getWatchList`             | Views addresses to watch minimum balance and how much to top it up.                                                                 |
| `getAccountInfo`           | Provides information about the specific target address, including the last time it was topped up. This function is _external only_. |

### `setWatchList` Function

#### Parameters

| Name              | Description                           | Suggested Setting           |
| ----------------- | ------------------------------------- | --------------------------- |
| `addresses`       | The list of addresses to watch        | (not applicable)            |
| `minBalancesWei`  | The minimum balances for each address | 5000000000000000000 (5 ETH) |
| `topUpAmountsWei` | The amount to top up each address     | 5000000000000000000 (5 ETH) |

Only the owner can `setWatchList`. Each of the parameters should be set with distinct requirements for each address.

### `setKeeperRegistryAddress` Function

#### Parameters

| Name                    | Description                                        |
| ----------------------- | -------------------------------------------------- |
| `keeperRegistryAddress` | Address that requires updating in `KeeperRegistry` |

Only the `keeperRegistryAddress` can `performUpkeep`, which is a _global setting_. `KeeperRegistry` addresses can be found on the [Chainlink Automation app](https://automation.chain.link/). However, only the owner can set a new `KeeperRegistry` after deployment.

### `setMinWaitPeriodSeconds` Function

#### Parameters

| Name     | Description                                                    | Suggested Setting |
| -------- | -------------------------------------------------------------- | ----------------- |
| `period` | Minimum wait period (in seconds) for addresses between funding | 3600 (1 hour)     |

`period` denotes the length of time between top ups for a specific address. This is a _global setting_ that prevents draining of funds from the contract if the private key for an address is compromised or if a gas spike occurs. However, only the owner can set a different minimum wait period after deployment.

### `topUp` Function

#### Parameters

| Name           | Description                                                |
| -------------- | ---------------------------------------------------------- |
| `needsFunding` | List of addresses to fund (addresses must be pre-approved) |

Any address can trigger the `topUp` function. This is an intentional design pattern that shows how easy it is to make an existing contract compatible with Chainlink Automation while maintaining an open interface. All validations are performed before the funding is triggered. If the conditions are not met, any attempt to top up will revert.
