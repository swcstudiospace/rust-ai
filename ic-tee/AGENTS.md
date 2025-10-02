# IC-TEE Agent Handbook

## Overview
- **Purpose**: Integrate Trusted Execution Environments (TEEs) with the Internet Computer (IC), providing SDKs, CLIs, and services for attestation, gateway access, and enclave orchestration.
- **Deliverables**: Rust crates for canisters/clients/daemons, Nitro enclave tooling, and docs guiding secure deployments.
- **Consumers**: Anda framework/cloud, IC-OSS, dMsg, and other IC dApps requiring confidential compute.

## Workspace Map
| Crate | Description | Notes |
| --- | --- | --- |
| `ic_tee_agent` | Client library interacting with IC TEE gateway/canisters | Used by `anda_engine`, `anda_cloud` |
| `ic_tee_cdk` | Canister Development Kit utilities for TEE-aware canisters | Provides guards, attestation verification |
| `ic_tee_cli` | CLI for provisioning, managing TEEs | Wraps agent + host APIs |
| `ic_tee_daemon` | Enclave-side daemon handling attestation and gateway communication | Runs inside Nitro enclave |
| `ic_tee_host_daemon` | Host-level daemon managing enclave lifecycle | Interfaces with AWS Nitro CLI |
| `ic_tee_identity_canister` | On-chain identity/attestation registry | Provide DID documents & verification |
| `ic_tee_logtail` | Log streaming service for enclave diagnostics | Supports remote debugging |
| `ic_tee_nitro_attestation` | Library for parsing/validating AWS Nitro attestation docs | Reused across crates |
| `ic_tee_nitro_gateway` | Gateway service within Nitro enclave bridging external requests | Coordinates with host daemon |
| `nitro_enclave/` | Scripts/config for building and running Nitro images | Update when AWS CLI changes |

## Data & Control Flow
1. **Provisioning**: Host daemon launches Nitro enclave using configs under `nitro_enclave/`.
2. **Attestation**: Enclave daemon produces attestation document; `ic_tee_nitro_attestation` validates and sends to identity canister.
3. **Registration**: Identity canister stores attestation proofs, accessible to clients via `ic_tee_agent`.
4. **Gateway**: `ic_tee_nitro_gateway` processes incoming traffic, verifying payment/auth via IC canisters.
5. **Consumers**: Anda services verify TEEs before executing sensitive workloads.

## Development Workflow
| Task | Command |
| --- | --- |
| Build | `cargo build --workspace` |
| Test | `cargo test --workspace` |
| Format | `cargo fmt --all` |
| Lint | `cargo clippy --workspace --all-features` |
| Build enclave image | See `nitro_enclave/README.md`; typically `make image` |
| Run CLI | `cargo run -p ic_tee_cli -- --help` |

## Coding Guidelines
- Keep attestation logic auditable; include comments referencing AWS/IC specs.
- Use `thiserror` for descriptive error propagation across async boundaries.
- Ensure on-chain identity interfaces remain stable; coordinate with `anda-cloud` registry updates.
- For host/enclave daemons, prefer async runtime where feasible, but respect Nitro limitations.

## Integration Notes
- **Anda Engine**: Uses `ic_tee_gateway_sdk` (re-exported from this repo) to interact with TEEs; maintain compatibility when updating enums/structs.
- **Anda Cloud**: Gateway/payment services rely on identity canister contracts; document API changes in `README` and `docs/`.
- **IC-OSS**: May store attestation artifacts; coordinate on schema if modifications occur.

## Testing Tips
- Unit-test cryptographic verification using sample attestation documents in fixtures.
- Provide integration tests that mock IC calls (using `ic-agent` test utilities).
- For Nitro workflows, use AWS-provided sample attestation to avoid requiring live enclave during CI.

## Release Checklist
1. Update crate versions and changelog.
2. Run full workspace tests and linting.
3. Validate Nitro build scripts against latest AWS CLI (update docs if needed).
4. Coordinate release with dependent repos (`anda`, `anda-cloud`) when API changes occur.
