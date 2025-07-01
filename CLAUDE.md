# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository is a browser implementation project following the book『[作って学ぶ]ブラウザのしくみ』(ISBN: 978-4-297-14546-0) published by 技術評論社 (Gijutsu-Hyoron-sha).

### Book Information
- **Official Support Page**: https://gihyo.jp/book/2024/978-4-297-14546-0/support
- **Reference Implementation**: https://github.com/d0iasm/sababook
- **Errata and Supplements**: https://lowlayergirls.github.io/wasabi-help/

### Current Progress
- Chapter 2: URL parsing implementation (completed)
  - `saba_core/src/url.rs` - URL struct with parsing methods
  - Supports only HTTP scheme
  - Includes comprehensive unit tests

## Build Commands

### Standard Rust Build
```bash
# Build all crates
cargo build

# Run tests
cargo test --all

# Run tests for saba_core specifically
cargo test -p saba_core

# Build for no_std environment (wasm32)
cd saba_core
cargo build --target wasm32-unknown-unknown -Z build-std=core,alloc
```

### Wasabi OS Build (for later chapters)
```bash
# Build for Wasabi OS environment
cargo build --features=wasabi --target=x86_64-unknown-none
```

## Architecture

### Crate Structure
- **`saba`**: Main binary crate (currently for Wasabi OS hello world)
- **`saba_core`**: Core browser logic library (no_std environment)
  - URL parsing (Chapter 2)
  - HTTP client (Chapter 3 - to be implemented)
  - HTML parser & DOM (Chapter 4 - to be implemented)
  - CSS parser & layout (Chapter 5 - to be implemented)
  - Renderer (Chapter 6 - to be implemented)
  - JavaScript engine integration (Chapter 7 - to be implemented)

### Key Design Decisions
1. **no_std Environment**: The project uses `#![no_std]` with `alloc` for embedded/OS development compatibility
2. **Rust Toolchain**: Uses `nightly-2024-01-01` as specified in `rust-toolchain.toml` - DO NOT change this version
3. **Wasabi OS Integration**: Uses `noli` crate from the Wasabi OS project for system APIs

## Development Guidelines

### Important Rules
1. **Follow the book's implementation strictly** - Do not optimize or modify the approach unless fixing bugs
2. **Maintain toolchain version** - The project requires `nightly-2024-01-01` for specific features
3. **Progressive implementation** - Implement features chapter by chapter as per the book
4. **Test coverage** - Write tests as demonstrated in the book

### Chapter Implementation Status
- [x] Chapter 1: Environment setup
- [x] Chapter 2: URL parsing
- [ ] Chapter 3: HTTP client
- [ ] Chapter 4: HTML parser & DOM tree
- [ ] Chapter 5: CSS parser & style calculation
- [ ] Chapter 6: Layout & rendering
- [ ] Chapter 7: JavaScript support

### Code Style
- Use `rustfmt` for formatting
- Follow the book's naming conventions
- Keep implementations simple and educational rather than optimized

## CI/CD

GitHub Actions workflow is configured to:
- Run tests on push/PR to main branch
- Check code formatting with rustfmt
- Run clippy lints
- Verify no_std compatibility for wasm32 target

The workflow respects `rust-toolchain.toml` settings automatically.