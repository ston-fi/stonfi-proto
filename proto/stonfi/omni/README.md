# Omniston Protobuf API

This directory contains protobuf schemas for the STON.fi Omniston protocol.

The Omniston schema is versioned. The current version in this repository is:

- `v1beta8`

Older versions may remain in the repository for compatibility and migration
purposes, but new Omniston API work should be evaluated against the current
version first.

## Documentation

Higher-level Omniston documentation is published on `docs.ston.fi`.

- Overview: <https://docs.ston.fi/developer-section/omniston/overview>
- SDK: <https://docs.ston.fi/developer-section/omniston/sdk>
- Swap protocol: <https://docs.ston.fi/developer-section/omniston/swap-protocol>
- Resolvers: <https://docs.ston.fi/developer-section/omniston/resolvers>
- Integrator fees: <https://docs.ston.fi/developer-section/omniston/referral-fees>
- Local migration guide: `v1beta8/v1beta7-to-v1beta8-migration-guide.md`

## Directory Layout

- `v1beta8/` — current Omniston protobuf API version

Within a version directory, schemas are typically organized into:

- `types/` — shared protocol data structures;
- `trader/` — trader-facing API messages;
- `resolver/` — resolver-facing API messages.

## Notes

This README is the entry point for Omniston schemas in this repository.
