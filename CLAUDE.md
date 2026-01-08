# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains Claude Code skills for implementing hexagonal architecture (ports and adapters pattern) in Rust projects. It provides two complete skill implementations:

1. **implementing-hexagonal-actix** - For Actix-web framework
2. **implementing-hexagonal-axum** - For Axum framework

Each skill follows progressive disclosure design with a main SKILL.md file (<250 lines) and supporting reference files (CONCURRENCY.md, IMPLEMENTATION.md, TESTING.md).

## Architecture

Each skill directory contains:

- **SKILL.md** - Quick start guide and overview (primary entry point, loaded first)
- **IMPLEMENTATION.md** - Complete layer-by-layer implementation guide (loaded on demand)
- **CONCURRENCY.md** - Thread-safe patterns (Arc, Mutex, RwLock, channels) (loaded on demand)
- **TESTING.md** - Testing strategies and security best practices (loaded on demand)

The skills teach a three-layer hexagonal architecture:

```
domain/           # Framework-agnostic business logic
├── entities/     # Core business objects with validation
└── repositories/ # Repository trait definitions (ports)

application/      # Use cases and orchestration
├── use_cases/    # Application use cases
└── ports/        # Service interfaces

infrastructure/   # Framework-specific adapters
├── web/          # Actix/Axum handlers and routes
└── persistence/  # Database implementations
```

## Skill Standards Compliance

Both skills follow official Claude Code skill standards:

- SKILL.md files are under 500 lines (217-234 lines)
- Progressive disclosure with supporting files loaded only when needed
- Gerund naming convention (implementing-hexagonal-*)
- Specific descriptions with clear trigger terms
- Security-first approach with input validation and SQL injection prevention
- No comments in generated code examples
- Front matter with name, description, and allowed-tools

## Key Design Principles

### Progressive Disclosure
Skills are designed for optimal context usage. The SKILL.md provides quick-start patterns. Supporting files are referenced but only loaded when users need deeper guidance on concurrency, full implementation details, or testing.

### Framework Independence
Domain layer code is identical between Actix and Axum skills. Only the infrastructure/web layer differs, demonstrating the power of hexagonal architecture for swappable frameworks.

### Security First
All examples include:
- Input validation at domain boundaries
- Parameterized SQL queries to prevent injection
- Error sanitization
- Authentication/authorization patterns
- Rate limiting and request size limits

### Concurrency Patterns
Comprehensive coverage of Rust async/sync patterns:
- Arc for shared ownership
- Mutex/RwLock for thread-safe state
- tokio::sync primitives for async contexts
- parking_lot for high-performance locking
- Channel-based message passing

## Making Changes

When modifying skills:

1. **Keep SKILL.md concise** - Under 500 lines (current: 217-234 lines). Move detailed content to supporting files.

2. **Maintain consistency** - Domain and application layer code should be identical between Actix and Axum skills. Only infrastructure/web differs.

3. **Update both skills** - If you change shared patterns (domain entities, repositories, use cases), apply to both implementing-hexagonal-actix and implementing-hexagonal-axum.

4. **Preserve front matter** - Always keep the YAML front matter with name, description, and allowed-tools.

5. **Flexible dependencies** - Use flexible version requirements (e.g., `tokio = "1"`) rather than strict locks to stay current with the ecosystem.

6. **Test code examples** - All Rust code in skills should compile. Examples should be production-ready, not pseudocode.

## Common Dependencies Referenced

Skills reference these common Rust dependencies:

Core:
- tokio - Async runtime with features: ["full"]
- async-trait - Async trait support
- uuid - Unique identifiers with features: ["v4", "serde"]
- serde, serde_json - Serialization
- sqlx - Type-safe SQL with features: ["runtime-tokio-native-tls", "postgres", "uuid"]

Concurrency:
- parking_lot - High-performance synchronization primitives

Actix-specific:
- actix-web - Web framework

Axum-specific:
- axum - Web framework
- tower - Middleware composition
- tower-http - HTTP middleware with features: ["cors", "trace"]

## Installation Patterns

Skills are designed for two installation modes:

Personal (~/.claude/skills/):
- Available across all projects for the user
- Recommended for developers frequently building hexagonal Rust services

Project-specific (.claude/skills/):
- Available only in specific project
- Recommended for team sharing or project-specific customizations
