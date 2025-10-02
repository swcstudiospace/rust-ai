# Anda DB Agent Handbook

## Mission
- **Purpose**: Provide the cognitive memory layer for Anda agents, combining document storage, graph reasoning (KIP), and vector search.
- **Outputs**: Rust crates (`anda_db`, `anda_cognitive_nexus`, `anda_kip`, etc.), capsules utilities, macros, and docs consumed by `anda`, `anda-cloud`, and partner products.

## Workspace Structure
| Crate | Description | Key Dependencies | Downstream Consumers |
| --- | --- | --- | --- |
| `anda_db` | Core database engine (collections, query execution, index orchestration) | `object_store`, `tokio`, `serde` | `anda_engine`, `anda_cloud` |
| `anda_cognitive_nexus` | High-level KIP execution engine (KQL/KML orchestration) | `anda_db`, `anda_kip` | `anda_engine` |
| `anda_kip` | Parser & executor for the Knowledge Interaction Protocol | `nom`, `serde`, `thiserror` | Anda framework, KIP tooling |
| `anda_db_schema` | Schema definitions & builders for collections | `serde`, `schemars`, `anda_db_derive` | `anda_core`, `anda_engine` |
| `anda_db_derive` | Procedural macros (e.g., `AndaDBSchema`) | `syn`, `quote` | Schema crates |
| `anda_object_store` | Wrapper over `object_store` with metadata/encryption | `aes-gcm`, `tokio`, `serde` | `anda_db`, `ic-oss`, `anda` |
| `anda_db_btree` | B-Tree index implementation | `serde`, `parking_lot` | `anda_db` |
| `anda_db_hnsw` | Approximate NN vector index (HNSW) | `rand`, `serde` | `anda_db` |
| `anda_db_tfs` | Full-text BM25 search | `tantivy`, `serde` | `anda_db` |
| `anda_db_utils` | Shared helpers & extensions | multiple | All crates |

## Data Architecture
1. **Object Store**: All data/indices stored via `anda_object_store`, enabling local disk or cloud (S3, GCS) backends with optional encryption.
2. **Collections**: Created via `anda_db::collection::Collection`, configured with indexes defined by schema metadata.
3. **Indexes**: B-Tree / BM25 / HNSW maintained incrementally; hybrid search merges results using Reciprocal Rank Fusion.
4. **KIP Layer**: `anda_kip` parses KML/KQL; `anda_cognitive_nexus` translates into DB operations and maintains graph metadata.
5. **Capsules**: KIP capsules (from `KIP/` repo) can be applied to seed domain knowledge; tests ensure compatibility.

## Development Workflow
- **Build & test**
  ```bash
  cargo fmt --all
  cargo test --workspace
  ```
- **Benchmarks** (vector search, indexing)
  ```bash
  cargo bench -p anda_db_hnsw
  ```
- **Docs**
  ```bash
  cargo doc -p anda_kip --open
  ```
- **Example Runs**
  ```bash
  cargo run -p anda_cognitive_nexus --example simple_kip
  ```

## Coding Standards
- Keep public APIs documented; align naming with KIP spec (Concept, Proposition, Domain).
- Capture detailed error contexts via `thiserror` enums to aid consuming agents.
- Avoid blocking IO in async functions; rely on `tokio` and `async-stream` utilities.
- Ensure `serde` derives maintain backward compatibility; use feature flags when introducing breaking schema changes.

## Integration Checklist
- Changes to KIP grammar must sync with `KIP/README` and `anda` orchestrator tests.
- When modifying encryption or object store behavior, update `ic-oss` notes and `anda_core` wrappers.
- Provide migration paths for index/schema changes (scripts or docs in `docs/`).
- For new crate releases, bump versions consistently across workspace and downstream `Cargo.toml` files.

## Testing Tips
- Use `cargo test -p anda_kip` for parser coverage; include sample `.kip` fixtures.
- Integration tests should open a temporary object store (see `MetaStoreBuilder` usage) to validate persistence.
- For performance validation, run the benches under `cargo bench` and capture metrics in PR descriptions.

## Release Checklist
1. Update crate versions + changelog entries.
2. Run full workspace tests/benches.
3. Sync with `anda` repo to ensure dependency bumps land simultaneously.
4. Publish crates to crates.io in dependency order (`anda_object_store` → indexes → `anda_db` → `anda_cognitive_nexus` → `anda_kip`).
