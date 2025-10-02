# KIP Agent Handbook

## Overview
- **Project goal**: Define the Knowledge Interaction Protocol (KIP) that bridges LLM "neural cores" with persistent knowledge graphs.
- **Deliverables**: Formal specification documents, executable capsules, system instructions, and tool schemas consumed by the Anda ecosystem.
- **Downstream consumers**: `anda-db` (implements the runtime), `anda`/`anda-app` (ships capsules & instructions), any MCP/LLM agent integrating KIP tooling.

## Repository Layout
| Path | Description | Notes |
| --- | --- | --- |
| `README.md`, `README_CN.md` | Specification primer in EN/ZH | Maintain consistency when updating examples or syntax |
| `SystemInstructions.md` | Guidance for LLM agents executing KIP | Used verbatim inside Anda agents and MCP tools |
| `FunctionDefinition.json` | JSON schema describing the `execute_kip` function/tool | Maps 1:1 with OpenAI/AWS function-call envelopes |
| `capsules/Genesis.kip` | Bootstraps the Domain, ConceptType, PropositionType meta-graph | Must be applied before other domain capsules |
| `capsules/Event.kip` | Defines situational memory type | Keeps event provenance consistent across products |
| `capsules/Person.kip` | Canonical Person concept/proposition schema | Referenced by identity onboarding flows |
| `capsules/persons/` | Example Personas (`system.kip`, `self.kip`) | Serve as reference prompts for new deployments |

## Knowledge Capsule Lifecycle
1. **Authoring**: Write `.kip` files in declarative KML/KQL form. Keep naming aligned with the conventions in `README` (CamelCase Types, snake_case proposition handles).
2. **Validation**: Run capsules through the Anda SDK (`anda_kip` crate) or via `anda-db` tests to ensure parsing succeeds.
3. **Bootstrapping**: Apply `Genesis.kip` first, then domain-specific capsules (Event, Person, etc.). Ensure downstream repos load them in this order.
4. **Versioning**: Increment the spec history table in `README.md` whenever syntax/rules change. If capsules evolve, add migration notes to downstream repos (`anda-db/docs`, `anda-app/src-tauri/kip`).

## Integration Touchpoints
- **Anda DB**: Implements KIP parsers/executors (`rs/anda_kip`). Any schema change here must be reflected in the spec and vice versa.
- **Anda App**: Bundles capsules for offline initialization (`src-tauri/kip`). Keep file names consistent to avoid installer regressions.
- **FunctionDefinition.json**: Imported by Anda and Forge/Codex tools to expose KIP execution; maintain backwards compatibility when extending parameters.

## Authoring Guidelines
- Prefer multi-line commands (`UPSERT`, `DELETE`, `FIND`) with clear indentation matching README examples.
- Use descriptive proposition handles (`?agent`, `?event`) to keep queries readable.
- Whenever adding a new capsule, document:
  1. Intent/purpose
  2. Dependencies (e.g., requires Genesis, Person)
  3. Expected execution order
- Provide English comments inline using `#` so agents can surface tooltips.

## Testing & Validation
- Leverage the Anda DB workspace:
  ```bash
  cd ../anda-db
  cargo test -p anda_kip
  ```
- For manual smoke tests, run the `execute_kip` function through Anda CLI once integration is wired.

## Change Control
- Coordinate spec updates with the `anda` and `anda-db` teams. Breaking changes require synced PRs across the stack.
- Update both `README.md` and `README_CN.md` when terminology or syntax shifts.
- Keep capsule changes atomic and reversible; downstream systems may need migration scripts.
