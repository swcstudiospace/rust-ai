# Rust Kanban Agent Handbook

## Overview
- **Purpose**: Terminal-based Kanban application built with Ratatui, offering mouse/keyboard interaction, cloud saves, encryption, and theming.
- **Executables**: Single binary `rust-kanban` interactive TUI.
- **Key Systems**: Application state management, IO (local/cloud persistence), TUI rendering, input handling, command palette, logging.

## Module Map (`src/`)
| Module | Responsibility | Notes |
| --- | --- | --- |
| `app/` | Core domain state (boards, cards, tags, history) and business logic | Provides undo/redo, serialization, commands |
| `inputs/` | Keyboard/mouse input handling, command palette bindings | Maps events to app actions |
| `ui/` | Ratatui UI widgets, layout, rendering pipeline | Subfolders for rendering primitives, widgets, text box |
| `ui/rendering/` | Rendering helpers (board views, dialogs, modals) | Keep styles consistent |
| `ui/text_box/` | Shared text input widget with selection support | Used across dialogs |
| `ui/widgets/` | Custom widgets (command palette, toasts, logs) | Built on Ratatui components |
| `io/` | Persistence (local filesystem, cloud sync, encryption) | Handles Supabase/HTTP interactions, config paths |
| `main.rs` | CLI parsing (clap), environment setup, logging bootstrap | Launches async runtime |

## Data Flow
1. **Startup**: CLI parses flags (config location, encryption key, logging). App loads config and state via `io` module.
2. **Event Loop**: Tokio runtime polls terminal events (crossterm) and input handlers translate into `AppAction`s.
3. **State Updates**: `app` module mutates board/card state, pushes undo history, and triggers persistence when needed.
4. **Rendering**: UI modules render current state into Ratatui frames, applying themes and layout.
5. **Persistence**: `io` module writes local JSON snapshots, coordinates cloud API for sync, handles AES-GCM encryption.

## Development Workflow
| Task | Command |
| --- | --- |
| Format | `cargo fmt` |
| Lint | `cargo clippy --all-targets` |
| Test | `cargo test` (limited coverage; consider adding) |
| Run | `cargo run --release -- --config <path>` |
| Logging | Controlled via environment/config; use `bunt` for colored logs |

## Coding Guidelines
- Prefer `parking_lot::Mutex` for shared state to minimize poisoning.
- UI code should use Ratatui stylize helpers (align with Codex/Forge conventions).
- Keep input mappings centralized (`inputs/`) to avoid conflicting bindings.
- For new persistence backends (BYODB), extend `io` with trait-based adapters.
- Wrap network calls (Supabase) with retry/backoff; ensure async functions propagate `eyre::Result`.
- Maintain theme definitions and handle dark/light palettes in `ui`.

## Feature Integration
- **Cloud Saves**: `io::cloud` handles encryption keys; ensure instructions warn users to back up keys.
- **Command Palette**: keep new actions discoverable; update palette entries when adding features.
- **Undo/Redo**: When mutating state, update history stacks and ensure actions are reversible.
- **Theming**: Add new themes under config handling; update README screenshots if visuals change.

## Testing Tips
- Limited automated tests; prioritize manual smoke tests:
  1. Launch app, create/move cards/boards.
  2. Test save/load cycles (local + cloud).
  3. Verify command palette entries render and execute.
  4. Check encryption flow using `--encryption-key` flag.
- Consider adding snapshot tests for serialized state to increase coverage.

## Release Checklist
1. Update version in `Cargo.toml` + changelog (`Changelog.md`).
2. Regenerate screenshots/gifs if UI changed.
3. Run `cargo fmt`, `cargo clippy`, and release build (`cargo build --release`).
4. Smoke-test on at least one platform; collect feedback from macOS testers if possible.
5. Publish to crates.io (`cargo publish`) and attach binaries to GitHub release.
