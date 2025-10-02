# Anda Cloud Agent Handbook

## Overview
- **Mission**: Deliver decentralized infrastructure (ICP + TEE) for registering, discovering, paying, and bridging Anda agents.
- **Components**: ICP canisters (Rust), TEE services, TypeScript web dashboard, shared CDK utilities.
- **Consumers**: `anda` engine (for registry/gateway integration), external agent marketplaces, Anda App dashboard.

## Repository Structure
| Path | Description | Notes |
| --- | --- | --- |
| `rs/anda_cloud_cdk/` | Shared types, guards, and utilities for writing Anda cloud canisters | Re-exported into multiple canisters and `anda` crates |
| `rs/anda_registry_canister/` | Main on-chain registry for agent metadata, handles, attestation | Includes Candid + integration tests |
| `rs/anda_discovery_service/` | (WIP) TEE-assisted discovery microservice | Aligns with MCP/A2A protocols |
| `rs/anda_payment_service/` | (WIP) Settlement engine implementing X402 & multi-token support | Integrates with TEE attestation and ledger canisters |
| `rs/anda_gateway_service/` | (WIP) Gateway bridging external agents into Anda network | Must validate identity + payments |
| `ts/anda_cloud_website/` | Next.js front-end for marketplace/management | Uses generated declarations for on-chain interaction |
| `declarations/` | Auto-generated Candid bindings (TS & Rust) | Regenerate with `dfx generate` after interface changes |
| `dfx.json`, `canister_ids.json` | ICP deployment configuration | Keep environment-specific IDs separate when possible |

## Data Flow
1. **Registration**: Agents call `anda_registry_canister` (Candid API) to register handles, attestation proofs, endpoints.
2. **Discovery (TEE)**: Discovery service queries registry + health checks via TEE, caches enriched metadata.
3. **Payments**: Payment service coordinates escrow/payments, exposes API integrated with gateway & registry.
4. **Gateway**: Bridges external protocols (MCP/A2A) by validating identity, routing traffic, and optionally invoking payments.
5. **Website**: Uses declarations to display registry state, configure agents, manage payments.

## Development Commands
| Task | Command |
| --- | --- |
| Install toolchain | `dfx deps install` + `cargo install ic-utils` (if needed) |
| Build Rust canisters | `cargo build --release -p anda_registry_canister` |
| Run tests | `cargo test --workspace` (Rust) / `pnpm test` (website) |
| Start local replica | `dfx start --background` |
| Deploy locally | `dfx deploy` |
| Generate declarations | `dfx generate` |
| Build web | `cd ts/anda_cloud_website && pnpm build` |

## Coding Standards
- Keep Candid interfaces in `src/lib.rs` (or `candid.did`) consistent with generated bindings; enforce via tests in `rs/anda_registry_canister/tests`.
- Use `ic_cdk` patterns for canister entrypoints; centralize guards (auth/access) inside the CDK helper crate.
- For TEE services, integrate `ic_tee_gateway_sdk` and follow attestation workflows defined in `ic-tee` repo.
- Document state machine transitions (registration status, payment lifecycle) in `docs/` when they change.

## Integration Touchpoints
- **Anda Engine**: uses registry for discovery; ensure API remains stable or provide migration guidelines.
- **Anda App**: front-end consumes the website’s APIs and declarations; coordinate UI contract changes.
- **IC-TEE**: gateway & payment services rely on TEE attestation libraries; update both repos in lockstep.
- **IC-OSS**: large artefacts (contracts, proofs) may be stored in OSS buckets; provide hooks if needed.

## Release Checklist
1. Run workspace tests (`cargo test --workspace`, `pnpm test`).
2. Bump versions in `Cargo.toml` and `package.json` where applicable.
3. Update `canister_ids.json` only for deployed environments; avoid committing local IDs.
4. Tag release with matching Anda framework/app updates if API changes occur.
