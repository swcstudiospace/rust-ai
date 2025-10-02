# Anda Agent Handbook

## Mission & Scope
- **Purpose**: Provide a Rust-first framework for composing autonomous agents with persistent memory (KIP) and secure execution (TEE/ICP).
- **Outputs**: Core crates (`anda_core`, `anda_engine`, ...), reference agents, tooling, and docs consumed by `anda-app`, `anda-cloud`, and partner deployments.
- **Key dependencies**: `anda-db` (memory), `ic-tee` (secure compute), `ic-oss` (object storage), `KIP` capsules.

## Workspace Topology
| Path | Role | Notable Dependencies |
| --- | --- | --- |
| `anda_core/` | Shared types (agents, workflows, envelopes), trait definitions, error taxonomy | `anda_db_schema`, `ic_*` crates, `object_store` |
| `anda_engine/` | Runtime for scheduling agent tasks, invoking tools, persisting memory | `anda_core`, `anda_db`, `anda_kip`, `ic_tee_*`, `anda_object_store` |
| `anda_engine_server/` | Axum-based HTTP layer hosting multiple engines, exposes REST/MCP endpoints | `anda_engine`, `ic_tee_agent`, `structured-logger` |
| `anda_cli/` | Command-line interface for provisioning, running engines, and managing capsules | Talks to engine server & cloud services |
| `anda_web3_client/` | Non-TEE client SDK for integrating agents with ICP/Web3 services | `ic-agent`, `ic_tee_gateway_sdk` |
| `agents/` | Ready-made agent definitions (pipelines, prompts, workflows) | Compose `anda_engine` traits + characters |
| `characters/` | Narrative/role profiles used by sample agents | Linked from `agents/` configs |
| `examples/` | End-to-end demos (ICP ledger agent, etc.) showing integration patterns | Use CLI, engine, and cloud crates |
| `tools/` | Domain-specific toolkits (ICP, BNB connectors, etc.) packaged as crates | Implement Tool traits from `anda_core` |
| `docs/` | Architecture, protocol references, diagrams | Keep synced with implementation changes |

## Runtime Flow
1. **Agent Configuration** (YAML/JSON) is parsed into `anda_core` domain types.
2. **Engine** loads tools, connects to `anda_cognitive_nexus` (from `anda-db`) using configured object store and encryption policies.
3. **TEE/ICP Integration**: Requests may route through `ic_tee_gateway_sdk` for attestation and `ic_auth_verifier` for cryptographic validation.
4. **Memory Operations**: Agent plans are persisted via KIP commands executed through the cognitive nexus.
5. **External Interfaces**: Engine server exposes HTTP endpoints (REST/MCP) consumed by `anda-app` or `anda-cloud` gateway.

## Development Tasks
- **Bootstrap**
  ```bash
  cargo fmt --all
  cargo test --workspace
  ```
- **Targeted testing**: Use `cargo test -p <crate>` to keep cycles fast. Engine has integration tests requiring local ICP env (see crate README).
- **Docs**: Update `docs/architecture.md` when altering call graphs or component responsibilities.
- **Examples**: Ensure example binaries compile (`cargo run -p <example-crate>`).

## Coding Standards
- Prefer builder-style APIs via `derive_setters` (mirroring Forge/Anda guidelines).
- Asynchronous boundaries use `async-trait`; avoid blocking calls inside executor-critical sections.
- When crossing network or TEE boundaries, surface explicit error contexts with `thiserror` enums in `anda_core`.
- Keep module naming consistent (e.g., `service`, `handler`, `domain`).

## Integration Checklist
- **Memory schema**: Verify new agent concepts have corresponding KIP capsule updates (`KIP/` repo) and DB schema definitions (`anda-db/rs/anda_db_schema`).
- **TEE changes**: Coordinate with `ic-tee` if attestation flows shift (update gateway SDK + docs).
- **Object storage**: Large artefacts should rely on `ic_oss_types`; re-exported via `anda_core` for type safety.
- **CLI/Server parity**: Whenever engine options change, update both `anda_cli` argument parsing and `anda_engine_server` config ingestion.

## Useful Commands
| Task | Command |
| --- | --- |
| Format entire workspace | `cargo fmt --all` |
| Lint targeted crate | `cargo clippy -p anda_engine --all-features` |
| Regenerate docs | `cargo doc --workspace --no-deps` |
| Run engine server | `cargo run -p anda_engine_server -- --config path/to/config.toml` |
| Launch sample agent | `cargo run -p anda_cli -- agent start --config agents/anda_bot/config.toml` |

## Coordination Notes
- Ship breaking changes via synchronized PRs across `anda`, `anda-db`, and `anda-cloud`.
- Maintain README translations (`README_CN.md`, `README_JA.md`) when core terminology changes.
- Keep characters/agents directories aligned with `anda-app` presets to avoid UX drift.
