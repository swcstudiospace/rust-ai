# IC-OSS Agent Handbook

## Overview
- **Purpose**: Provide decentralized object storage primitives on the Internet Computer (IC) with scalable buckets, clusters, and client SDKs.
- **Deliverables**: Rust canisters, Rust SDKs, TypeScript SDK, CLI, docs, and examples demonstrating storage flows.
- **Consumers**: Anda framework/cloud, dMsg, and external dApps requiring reliable storage.

## Repository Layout
| Path | Description | Notes |
| --- | --- | --- |
| `src/ic_object_store/` | Rust client SDK for interacting with object store canisters | Used by Anda DB/object store wrapper |
| `src/ic_object_store_canister/` | Canister implementation storing metadata/objects (Arrow-backed) | Shared by bucket/cluster logic |
| `src/ic_oss_bucket/` | Bucket smart contract handling storage operations | Exposes Candid API |
| `src/ic_oss_cluster/` | Cluster manager coordinating buckets, access control | Maintains sharding, permissions |
| `src/ic_oss_can/` | Helper crate for embedding IC-OSS in other canisters | Minimal runtime |
| `src/ic_oss_types/` | Shared types/enums across crates | Re-exported by `anda_core` |
| `src/ic_oss/` | High-level Rust SDK for clients | Wraps object_store + types |
| `src/ic_oss_ts/` | TypeScript SDK for web clients | Published to npm |
| `src/ic_oss_cli/` | Rust CLI for managing buckets/uploads | Useful for testing |
| `examples/` | Sample canisters/web apps (AI canister, upload JS, video player) | Keep in sync with API changes |
| `docs/` | Access control, architecture diagrams | Update with feature changes |

## Data Flow
1. **Cluster Setup**: Admin configures cluster via `ic_oss_cluster` canister (bucket mapping, access policies).
2. **Client Auth**: Front-end obtains access token using `ic_oss_ts` or Rust SDK, honoring bucket permissions.
3. **Upload**: Files are chunked, optionally encrypted, and stored via bucket canister APIs. Sharding handled automatically.
4. **Download**: Clients request segments; bucket streams data while respecting access rules.
5. **Integration**: Applications like Anda or dMsg store long-term artifacts (memory snapshots, user files) via these APIs.

## Development Workflow
| Task | Command |
| --- | --- |
| Build workspace | `cargo build --workspace` |
| Run tests | `cargo test --workspace` |
| Format | `cargo fmt --all` |
| Lint | `cargo clippy --workspace --all-features` |
| Build TS SDK | `cd src/ic_oss_ts && pnpm install && pnpm build` |
| Deploy locally | `dfx deploy` (after `dfx start`) |

## Coding Guidelines
- Keep Candid interfaces documented; regenerate bindings after changes (`dfx generate`).
- Ensure encryption helpers default to secure configurations; document any optional plaintext modes.
- Propagate meaningful errors using `thiserror`; clients depend on descriptive error codes.
- Maintain compatibility between Rust and TS SDKs—update both when request/response structs change.

## Integration Touchpoints
- **Anda DB**: Uses `anda_object_store` built atop `ic_object_store`; coordinate API/metadata schema updates.
- **Anda Cloud**: Payment/gateway services may issue bucket credentials; keep access control docs accurate.
- **IC-TEE**: If enabling TEE-backed uploads, follow attestation flows defined there.

## Testing Tips
- Use `dfx start --background` + `dfx deploy` for local integration tests.
- Examples under `examples/` serve as smoke tests; run them after major API changes.
- Ensure CLI (`ic-oss-cli`) commands cover upload/download flows; add integration tests when introducing new flags.

## Release Checklist
1. Update version numbers across crates/SDKs.
2. Run workspace tests + TS build.
3. Refresh documentation (diagrams, README links).
4. Publish crates (`cargo publish`) and npm package (`pnpm publish`) in dependency order.
5. Announce breaking changes with migration steps (bucket schema, access policy updates).
