# Omniston Glossary


## Actors

* **Trader** — protocol participant that requests quotes, creates swaps or orders, and provides the **Input asset**.
* **Resolver** — protocol participant that provides quotes and settles trades by delivering the **Output asset**.
* **Protocol** — the Omniston system, including its API, routing logic, and settlement contracts.
* **Integrator** — external application or partner that brings flow into the protocol and may receive an integrator fee.

## Assets

* **Input asset** — asset provided by the **Trader** to start a trade.
* **Input amount** — amount of the **Input asset** involved in a quote and a trade.
* **Output asset** — asset the **Trader** expects to receive as the result of a trade.
* **Output amount** — amount of the **Output asset** involved in a quote and a trade.
* **Source blockchain** — blockchain on which the **Input asset** is provided and the trade is initiated.
* **Destination blockchain** — blockchain on which the **Output asset** is delivered to the **Trader**.

## RFQ Protocol

* **RFQ** — request for quote sent by the **Trader** to ask resolvers for trade terms.
* **Quote** — resolver-specific offer that defines trade amounts, fees, and settlement data for a given **RFQ**.
* **Swap settlement** — settlement mode in which the trade is executed immediately through one or more swap routes.
* **Order settlement** — settlement mode in which the trade is represented as an order or position that can be filled by a **Resolver**.

## Swap settlement

* **Swap route** — complete path used to convert the **Input asset** into the **Output asset**.
* **Swap step** — one asset-to-asset conversion inside a **Swap route**.
* **Swap chunk** — the smallest executable part of a **Swap step**, typically bound to a specific liquidity source or protocol.

## Order settlement

* **Order** — trade object created from a quote and exposed for filling, fully or partially, by one or more resolvers.
* **Signed Order** — blockchain-specific signed payload that authorizes settlement without first depositing funds into an escrow position.
* **Order details** — off-chain structure whose hash is embedded into settlement transactions to verify the trade intent.
* **Execution** — individual fill attempt or fill fragment within an **Order**.
* **Input position** — position on the **Source blockchain** that holds or represents the **Input asset** for an **Execution**.
* **Output position** — position on the **Destination blockchain** that holds or represents the **Output asset** for an **Execution**.

## Contracts and vaults

* **Protocol contract** — smart contract used by the protocol to enforce settlement rules on a blockchain.
* **Escrow factory contract** — contract that creates or manages escrow positions for **Order settlement**.
* **Escrow position contract** — contract instance that stores the assets and state of a particular escrow-backed position.
* **Vault contract** — contract that stores the funds of participants for later use in settlement flows.
* **Escrow vault** — vault managed by an **Escrow factory contract** that holds assets used in **Order settlement**.
* **Fee vault factory contract** — contract that creates or manages **Fee vaults**.
* **Fee vault** — vault that accumulates fees for a specific owner and asset.

## Operations

* **Order cancellation** — operation by which the **Trader** stops an **Order** and withdraws the remaining funds from the **Escrow position**.
* **Execution completion** — successful finalization of an **Execution**, in which the exchange is completed according to protocol rules.
* **Execution rollback** — unsuccessful finalization of an **Execution**, in which assets are returned according to protocol rules.
