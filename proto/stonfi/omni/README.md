# Omniston Protobuf API

This directory contains protobuf schemas for the STON.fi Omniston protocol and
history APIs.

The APIs are versioned independently. The current versions in this repository
are:

- Omniston protocol API: `v1beta8`
- Omniston History API: `history/v1`

Older versions may remain in the repository for compatibility and migration
purposes, but new API work should be evaluated against the corresponding
current version first.

## Documentation

Higher-level Omniston documentation is published on `docs.ston.fi`.

- Overview: <https://docs.ston.fi/developer-section/omniston/overview>
- SDK: <https://docs.ston.fi/developer-section/omniston/sdk>
- Swap protocol: <https://docs.ston.fi/developer-section/omniston/swap-protocol>
- Resolvers: <https://docs.ston.fi/developer-section/omniston/resolvers>
- Integrator fees: <https://docs.ston.fi/developer-section/omniston/referral-fees>
- Local migration guide: `v1beta8/v1beta7-to-v1beta8-migration-guide.md`
- Local History API schemas: `history/v1/`

## Directory Layout

- `v1beta8/` — current Omniston protocol API
- `history/v1/` — finalized-order history and aggregate statistics API

The protocol API is organized into:

- `types/` — shared protocol data structures;
- `trader/` — trader-facing API messages;
- `resolver/` — resolver-facing API messages.

The History API provides:

- `FinalizedOrdersRpc` — paginated finalized-order history and lookup by quote
  ID;
- `AggregatesRpc` — filtered and grouped finalized-order statistics.

## Notes

This README is the entry point for Omniston schemas in this repository.
