# Loom

**Repository:** [ghuntley/loom](https://github.com/ghuntley/loom)
**Stars:** 666
**Language:** Rust
**Status:** Research Project (Experimental)
**License:** Proprietary
**Source:** GitHub API

## CAUTION

Loom is a research project. If your name is not Geoffrey Huntley then do not use. This software is experimental, unstable, and under active development. APIs will change without notice. Features may be incomplete or broken. There is no support, no documentation guarantees, and no warranty of any kind. Use at your own risk.

## Overview

Loom is an AI-powered coding agent built in Rust. It provides a REPL interface for interacting with LLM-powered agents that can execute tools to perform file system operations, code analysis, and other development tasks.

The system is designed around three core principles:

1. **Modularity** - Clean separation between core abstractions, LLM providers, and tools
2. **Extensibility** - Easy addition of new LLM providers and tools via trait implementations
3. **Reliability** - Robust error handling with retry mechanisms and structured logging

## Architecture

Loom is organized as a Cargo workspace with 30+ crates:

### Key Components

- **Core Agent** - State machine for conversation flow and tool orchestration
- **LLM Proxy** - Server-side proxy architecture (API keys never leave the server)
- **Tool System** - Registry and execution framework for agent capabilities
- **Weaver** - Remote execution environments via Kubernetes pods
- **Thread System** - Conversation persistence with FTS5 search
- **Analytics** - PostHog-style product analytics with identity resolution
- **Auth** - OAuth, magic links, ABAC authorization
- **Feature Flags** - Runtime feature toggles, experiments, and kill switches

## Ralph Loop Orchestration

Loom serves as the orchestrator for "Ralph Loops" - a concept for autonomous agent execution and recursive task completion. The project focuses on building autonomous recursive outcomes where agents can deploy code with zero human code review.

## Innovation Focus

The creator emphasizes that the last 40 years of computing and software engineering practices were developed for humans. With new AI agents, many traditional practices can be invalidated. Loom is built to explore autonomous agent patterns beyond traditional software development workflows.

## Building

### With Nix (Preferred)

Uses cargo2nix for reproducible builds with per-crate caching:

```bash
nix build .#loom-cli-c2n
nix build .#loom-server-c2n
```

### With Cargo (Development)

```bash
cargo build --workspace
cargo test --workspace
cargo clippy --workspace -- -D warnings
cargo fmt --all
```

## Bookmarked By

@GeoffreyHuntley on January 15, 2026 - Released as open source to teach concepts and ideas around autonomous AI agent patterns and Ralph Loop orchestration. Author emphasizes this is a research exploration beyond traditional software engineering practices.
