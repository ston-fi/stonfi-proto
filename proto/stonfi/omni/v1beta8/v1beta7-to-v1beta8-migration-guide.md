# Omniston Migration Guide: `v1beta7` to `v1beta8`

This guide helps integrators understand what changed when moving from
Omniston `v1beta7` to `v1beta8` API.

It focuses on terminology, protocol concepts, and flow changes. It does not
attempt to document field-by-field changes or SDK migration details.

## Scope

Use this guide if you:

- work directly with Omniston protobuf schemas;
- operate resolver infrastructure;
- maintain backend integrations that reason about the protocol lifecycle;
- need to understand why `v1beta8` looks different even when the product goal
  is still "quote, create, and settle a trade".

If you integrate Omniston only through an SDK, treat this document as
a background context. The practical migration steps for SDK consumers should live
in a separate SDK-specific guide.

## Headline Changes

The main change in `v1beta8` is not a simple rename of methods or fields.
The API now models Omniston more explicitly as a protocol with distinct
objects and lifecycles.

In `v1beta7`, many integrations could think primarily in terms of:

- a quote;
- a trade;
- a stream of trade statuses;
- separate service surfaces for escrow and gasless flows.

In `v1beta8`, the preferred mental model is:

- an `RFQ` produces `Quote` updates;
- a quote can lead either to the immediate swap settlement or to order settlement;
- an `Order` is a persistent object with its own lifecycle;
- an order may have one or more `Execution` records;
- HTLC and intrachain flows are represented more explicitly.

This makes the API more expressive, but it also means clients should update
their terminology and flow assumptions rather than expecting a mechanical
one-to-one migration.

## Terminology Changes

The most visible migration work is terminology. `v1beta8` consistently uses
language that reflects protocol roles and asset direction more clearly.

### Asset Direction

`v1beta7` commonly used `bid` and `ask`.

`v1beta8` uses:

- `input`
- `output`

This is more than a naming preference. The new terms make the API read from
the trader's point of view:

- `input` is what the trader provides;
- `output` is what the trader receives.

For resolvers, `v1beta8` also makes a clearer distinction between:

- what the trader receives after fees;
- what the resolver must send including fees.

### Addresses and Asset IDs Are Now Different Types

`v1beta7` used a single generic `Address` type both for chain addresses and for
asset references such as `bid_asset_address` and `ask_asset_address`.

`v1beta8` splits these concerns explicitly:

- `ChainAddress` for wallet, trader, resolver, contract, and position
  addresses;
- `AssetId` for asset identifiers such as native assets, jettons, ERC-20, or
  ERC-1155 assets.

This is an important migration change, not just a schema cleanup.

In `v1beta7`, it was easy to treat "asset address" and "account or contract
address" as the same kind of value. In `v1beta8`, that assumption is no longer
correct:

- not every asset is represented by a plain chain address;
- native assets are represented as asset kinds, not contract addresses;
- asset validation and address validation should now be handled separately.

If your integration has shared helpers, database columns, API DTOs, or
validation logic for all "addresses", review them carefully. Many places that
stored a generic address in `v1beta7` should now store either a
`ChainAddress` or an `AssetId`, depending on the field semantics.

### Referral to Integrator

`v1beta7` used `referrer` and `referrer_fee_bps`.

`v1beta8` uses:

- `integrator`
- `integrator fees`
- `pips` instead of `bps`

This better matches how partner integrations are described in the protocol and
external documentation. If your internal systems still say "referral" or
"referrer", plan to align that language as part of migration.

This is not only a rename. The fee unit also changed:

- `v1beta7` commonly expressed fee configuration in `bps`;
- `v1beta8` uses `pips`.

If you have internal config, metrics, validation logic, or operator tooling
that assumes basis points, treat this as a semantic migration item rather than
as a cosmetic terminology update.

### Settlement Terminology

`v1beta7` exposed settlement primarily through `SettlementMethod` values such
as:

- `SWAP`
- `ESCROW`
- `HTLC`

`v1beta8` still distinguishes immediate swap settlement from order-based
settlement, but the API is framed less as "pick one transport enum" and more
as "work with a specific lifecycle":

- `Swap settlement`
- `Order settlement`

For migration purposes, the important shift is:

- `swap settlement` remains the immediate execution path;
- `order settlement` becomes the umbrella for flows that create an order or
  position first and settle it over time;
- HTLC semantics are now described as part of that order lifecycle instead of
  being treated only as an alternative terminal settlement method.

### Trade Status to Protocol Objects

`v1beta7` encouraged clients to reason through `TradeStatus` variants such as:

- `AwaitingTransfer`
- `Swapping`
- `AwaitingFill`
- `ClaimAvailable`
- `RefundAvailable`
- `TradeSettled`

`v1beta8` still exposes status, but the primary conceptual building blocks are
now explicit protocol objects:

- `Quote`
- `Order`
- `Execution`
- `Input position`
- `Output position`

This means downstream systems should shift from "current trade status enum" to
"which protocol object exists, and what phase is it in?".

## Package and API Surface Changes

The package namespace changed from `omni.v1beta7` to `stonfi.omni.v1beta8`.

This is not only a version bump. It reflects a cleaner separation between
legacy schema layout and the current STON.fi-owned public API surface.

On the trader side, `v1beta7` exposed several separate services, including:

- `QuoteGrpc`
- `TradeGrpc`
- `GaslessGrpc`
- `EscrowGrpc`
- `TransactionBuilderGrpc`

In `v1beta8`, the trader-facing surface is reorganized around clearer concerns:

- `QuoteRpc` for RFQ quote streams;
- `SwapRpc` for tracking swap settlement;
- `OrderRpc` for order creation, tracking, registration, and cancellation;
- chain-specific builder APIs under `trader/chains/`.

The resolver side is also cleaner:

- `ResolverGrpc` became `ResolverRpc`;
- quote handling is expressed through higher-level resolver quote objects;
- protocol-specific side channels are reduced in favor of explicit requests.

The result is a more cohesive API surface with fewer service boundaries tied to
legacy implementation details.

## Flow Changes

The biggest behavioral migration is how clients should think about lifecycle.

### 1. RFQ Is Now a More Explicit Entry Point

In both versions, integrations start with a request-for-quote flow.

In `v1beta8`, the RFQ model is more explicit and becomes the natural root for
the rest of the lifecycle:

- an RFQ produces quote updates;
- a quote may lead to the swap settlement or order settlement;
- the chosen settlement mode determines which API surface and lifecycle follows.

This sounds obvious, but it matters because `v1beta7` often encouraged clients
to jump mentally from quote directly to "trade tracking". `v1beta8` inserts a
clearer conceptual split between swap flow and order flow.

### 2. Swap Flow and Order Flow Are Separated

In `v1beta7`, a single trade-tracking model had to represent several very
different flows, including direct swap, escrow, and HTLC cases.

In `v1beta8`, these are separated more deliberately:

- `SwapRpc` handles immediate swap settlement progress;
- `OrderRpc` handles order-based settlement flows.

This is one of the most important migration points. Clients should no longer
assume that one generic "track trade" stream is the right abstraction for all
settlement modes.

### 3. Orders Are First-Class Protocol Objects

In `v1beta7`, order-related data existed, but the mental model was still
dominated by quote creation and trade status updates.

In `v1beta8`, an `Order` is a first-class object with:

- its own status;
- a persistent protocol address or contract context;
- cancellation semantics;
- one or more executions.

This is especially important for integrators that persist protocol state or
build operator tooling. You should now model orders as durable entities rather
than as a thin appendage to a quote.

### 4. Execution Is a Real Unit of Progress

`v1beta7` had partial-fill concepts, but they were not as central or explicit.

`v1beta8` introduces `Execution` as a clear unit of settlement progress. One
order may have several executions, especially when partial filling is involved.

This changes the right level of abstraction for monitoring and analytics:

- old mental model: "a trade is moving through statuses";
- new mental model: "an order may accumulate executions over time".

### 5. HTLC Flows Are More Explicit

`v1beta7` exposed HTLC-related milestones, but much of the protocol semantics
were implied by status names such as `ClaimAvailable` or `RefundAvailable`.

`v1beta8` makes HTLC concepts more explicit through dedicated types and order
execution data, including:

- hashing function;
- secret hashes;
- input and output positions;
- completion and rollback phases;
- secret disclosure.

The migration implication is that HTLC is no longer best understood as a
special branch of trade status handling. It should be understood as an explicit
protocol flow with its own positions, timing windows, and completion rules.

### 6. Intrachain and HTLC Flows Diverge More Clearly

In `v1beta7`, a lot of complexity was compressed into generic trade tracking.

In `v1beta8`, the distinction between intrachain and HTLC flows is much more
important to the shape of the data and to client logic.

Clients that reason below the SDK level should expect to branch earlier on:

- intrachain flow;
- HTLC flow.

That branching now affects how to interpret positions, timings, secrets, and
settlement progress.

## Gasless and Escrow Migration

One of the more visible structural changes is how gasless and escrow logic are
presented.

In `v1beta7`, there were dedicated trader-facing services for:

- gasless order operations;
- escrow order listing;
- transaction building.

In `v1beta8`, these concerns are reorganized around the order lifecycle.

This does not mean the underlying execution modes disappeared. Instead,
`v1beta8` represents them in a more unified way:

- gasless orders are part of the order flow;
- escrow-backed positions are also part of the order flow;
- chain-specific payload building is pushed into chain-specific APIs.

For migration, the practical takeaway is:

- do not look for a one-to-one replacement of every `GaslessGrpc` or
  `EscrowGrpc` method;
- instead, remap your integration around `OrderRpc` plus chain-specific order
  payload building where needed.

## Resolver Mental Model Changes

Resolver integrations should expect `v1beta8` to feel less like "submit a quote
for a settlement enum" and more like participation in a protocol with explicit
objects and constraints.

Important conceptual shifts:

- resolver quote data is more structured around source and destination roles;
- source-side and destination-side responsibilities are clearer;
- trade start deadlines, reservations, and order lifecycle matter more;
- HTLC-specific responsibilities are more explicit.

For resolver operators, this usually means revisiting:

- internal naming;
- quote construction logic;
- monitoring dashboards;
- fill, reservation, and cancellation workflows.

## What Usually Needs Renaming Internally

Even if your SDK shields you from most wire-level changes, internal language
often still leaks into dashboards, logs, metrics, and support runbooks.

These are the most common concept-level renames worth making:

- `bid` -> `input`
- `ask` -> `output`
- `referrer` -> `integrator`
- `referrer fee` -> `integrator fee`
- generic `trade tracking` -> `swap tracking` or `order tracking`, depending on
  the flow
- `gasless API` / `escrow API` as separate top-level concepts -> order flow
  variants within a single order lifecycle

Making these changes early reduces confusion when reading `v1beta8` comments,
docs, and support material.

## Suggested Migration Strategy

For most teams, the safest migration path is conceptual first, implementation
second.

1. Update terminology in internal documentation and code comments.
2. Separate your mental model into RFQ, swap flow, and order flow.
3. Treat orders as durable entities rather than temporary trade artifacts.
4. Treat executions as first-class progress units for order settlement.
5. Review any HTLC handling with the assumption that positions, phases, and
   secret disclosure are now explicit protocol concepts.
6. Only after that, map the concrete SDK or field-level changes.

This order usually produces fewer mistakes than trying to translate protobuf
symbols one by one.

## What This Guide Intentionally Does Not Cover

This document does not cover:

- field-by-field protobuf renames;
- exact message compatibility mappings;
- SDK method migrations;
- generated client code changes;
- chain-specific signing details.

Those topics should be documented separately where the target audience can act
on them directly.

## Summary

Migrating from `v1beta7` to `v1beta8` is primarily a migration of terminology
and flow model.

The key mindset shift is:

- stop thinking only in terms of quote plus trade status;
- start thinking in terms of RFQ, quote, swap or order settlement, order
  lifecycle, execution lifecycle, and explicit HTLC semantics.

Teams that adopt this conceptual model first will usually find the later SDK
and implementation migration much easier.
