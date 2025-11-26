# Proposal for Source Code Reorganization

This document outlines a plan to reorganize the `src/` directory to better reflect the application's architecture. The goal is to make the codebase easier to navigate and understand for new and existing developers.

## Motivation

The current structure of the `src/` directory is mostly flat. While this is simple, it doesn't expose the logical layers of the application. Key architectural components like the core data model, the view layer, and asynchronous services are intermingled.

By grouping files into directories based on their role, we can make the architecture explicit, reduce cognitive overhead, and make it easier to find relevant code.

## Guiding Principle

The proposed reorganization is based on the architectural analysis of the codebase, which identifies several distinct layers:

1.  **Core Model**: The pure data representation of a document.
2.  **Controller**: The orchestrator that manages models and connects all systems.
3.  **View & UI**: The presentation layer responsible for rendering.
4.  **Input Pipeline**: The system that translates user input into commands.
5.  **Services**: Asynchronous peripherals that communicate with the outside world.
6.  **Primitives**: Low-level utilities used by other layers.

The new directory structure will mirror these layers.

## Proposed Directory Structure

```
src/
├── app/                  // The main application controller and entry point
│   ├── mod.rs            // Formerly editor/mod.rs - the Editor struct
│   ├── input.rs
│   ├── render.rs
│   └── ...
├── model/                // The core data model for a "document"
│   ├── mod.rs
│   ├── state.rs
│   ├── buffer.rs         // Formerly text_buffer.rs
│   ├── piece_tree.rs
│   ├── cursor.rs
│   ├── event.rs
│   └── marker.rs
├── view/                 // UI components and rendering logic
│   ├── mod.rs
│   ├── split.rs
│   ├── viewport.rs
│   ├── popup.rs
│   ├── prompt.rs
│   └── ui/               // Existing ui/ directory moves here
├── input/                // The input-to-action-to-event pipeline
│   ├── mod.rs
│   ├── actions.rs
│   ├── commands.rs
│   ├── keybindings.rs
│   └── command_registry.rs
├── services/             // Asynchronous peripherals (LSP, plugins, FS)
│   ├── mod.rs
│   ├── lsp/
│   ├── plugins/
│   ├── fs/
│   └── async_bridge.rs
├── primitives/           // Low-level syntax and rendering utilities
│   ├── mod.rs
│   ├── highlighter.rs
│   ├── ansi.rs
│   ├── indent.rs
│   └── text.rs           // For word_nav, line_iter, etc.
├── lib.rs                // Crate root, re-exporting main components
└── main.rs               // Application entry point
```

## Reorganization Plan

This reorganization can be done incrementally.

### Phase 1: Create New Core Directories

Create the following new directories in `src/`:
- `src/app/`
- `src/model/`
- `src/view/`
- `src/input/`
- `src/services/`
- `src/primitives/`

### Phase 2: Relocate the Core Model

Move the files defining the pure document state into `src/model/`.

- **Move:**
  - `state.rs` -> `model/state.rs`
  - `text_buffer.rs` -> `model/buffer.rs`
  - `piece_tree.rs` -> `model/piece_tree.rs`
  - `cursor.rs` -> `model/cursor.rs`
  - `multi_cursor.rs` -> `model/multi_cursor.rs`
  - `marker.rs` -> `model/marker.rs`
  - `marker_tree.rs` -> `model/marker_tree.rs`
  - `event.rs` -> `model/event.rs`
  - `document_model.rs` -> `model/document_model.rs`
- **Action:** Create `src/model/mod.rs` to expose the public structs and update `use` paths across the codebase.

### Phase 3: Consolidate the Controller

Rename `src/editor/` to `src/app/` to better reflect its role as the central application controller.

- **Move:**
  - `editor/` -> `app/`
- **Action:** Update `use` paths. The `Editor` struct will now be `crate::app::Editor`.

### Phase 4: Relocate the Input Pipeline

Move the input handling chain into `src/input/`.

- **Move:**
  - `actions.rs` -> `input/actions.rs`
  - `commands.rs` -> `input/commands.rs`
  - `keybindings.rs` -> `input/keybindings.rs`
  - `command_registry.rs` -> `input/command_registry.rs`
  - `input_history.rs` -> `input/input_history.rs`
  - `position_history.rs` -> `input/position_history.rs`
- **Action:** Create `src/input/mod.rs` and update `use` paths.

### Phase 5: Relocate the View & UI Layer

Group all components responsible for presentation into `src/view/`.

- **Move:**
  - `ui/` -> `view/ui/`
  - `split.rs` -> `view/split.rs`
  - `viewport.rs` -> `view/viewport.rs`
  - `popup.rs` -> `view/popup.rs`
  - `prompt.rs` -> `view/prompt.rs`
  - `overlay.rs` -> `view/overlay.rs`
  - `virtual_text.rs` -> `view/virtual_text.rs`
  - `margin.rs` -> `view/margin.rs`
  - `file_tree/` -> `view/file_tree/`
  - `theme.rs` -> `view/theme.rs`
- **Action:** Create `src/view/mod.rs` and update `use` paths.

### Phase 6: Group Asynchronous Services

Group all modules that deal with external processes and I/O into `src/services/`.

- **Move:**
  - `lsp_manager.rs`, `lsp.rs`, `lsp_async.rs`, `lsp_diagnostics.rs` -> `services/lsp/`
  - `plugin_thread.rs`, `plugin_api.rs`, `plugin_process.rs`, `ts_runtime.rs` -> `services/plugins/`
  - `fs/` -> `services/fs/`
  - `async_bridge.rs` -> `services/async_bridge.rs`
  - `clipboard.rs` -> `services/clipboard.rs`
  - `signal_handler.rs` -> `services/signal_handler.rs`
- **Action:** Create `mod.rs` files for the new subdirectories and update `use` paths.

### Phase 7: Group Primitives & Utilities

Move the remaining low-level, reusable utilities into `src/primitives/`.

- **Move:**
  - `highlighter.rs`, `semantic_highlight.rs` -> `primitives/`
  - `ansi.rs`, `ansi_background.rs` -> `primitives/`
  - `indent.rs` -> `primitives/`
  - `text_property.rs` -> `primitives/`
  - `word_navigation.rs`, `line_wrapping.rs`, `line_iterator.rs` -> `primitives/text.rs` or a `primitives/text/` module.
  - `config.rs` -> could stay at root or move to `app/`. For now, leave at root.
- **Action:** Create `src/primitives/mod.rs` and update `use` paths.

## Follow-up Work

After each phase of moving files, a crucial step will be to update all `use` statements and module declarations (`mod.rs` and `lib.rs`) to reflect the new paths. This can be a tedious process but is essential for the compiler. Tools like `sed`, `grep`, or IDE-based find-and-replace will be necessary.

This phased approach allows the refactoring to be done in manageable chunks, reducing the risk of breaking the build for an extended period.
