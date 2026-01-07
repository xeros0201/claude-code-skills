# Claude Code Skills - Rust Hexagonal Architecture

Professional Claude Code skills for implementing hexagonal architecture (ports and adapters pattern) in Rust projects. Optimized for performance with progressive disclosure and comprehensive guides.

## Available Skills

### 1. Implementing Hexagonal Architecture with Actix-web
**Skill Name:** `implementing-hexagonal-actix`

Guides implementation of hexagonal architecture for Rust projects using Actix-web. Features progressive disclosure with concise quick-start and detailed reference files.

**Use When:**
- Building web services with Actix-web
- Need high-performance HTTP server
- Want mature, battle-tested framework
- Building applications with complex routing needs

**Structure:**
- `SKILL.md` - Quick start and overview (217 lines)
- `CONCURRENCY.md` - Thread-safe patterns (Arc, Mutex, RwLock, channels)
- `IMPLEMENTATION.md` - Complete layer-by-layer guide
- `TESTING.md` - Testing strategies and security practices

[View Skill](./implementing-hexagonal-actix/SKILL.md)

### 2. Implementing Hexagonal Architecture with Axum
**Skill Name:** `implementing-hexagonal-axum`

Guides implementation of hexagonal architecture for Rust projects using Axum. Features progressive disclosure with concise quick-start and detailed reference files.

**Use When:**
- Building web services with Axum
- Want compile-time route validation
- Prefer type-safe extractors and Tower ecosystem
- Need idiomatic Rust patterns
- Want smaller binary sizes

**Structure:**
- `SKILL.md` - Quick start and overview (234 lines)
- `CONCURRENCY.md` - Thread-safe patterns (Arc, Mutex, RwLock, channels)
- `IMPLEMENTATION.md` - Complete layer-by-layer guide
- `TESTING.md` - Testing strategies and security practices

[View Skill](./implementing-hexagonal-axum/SKILL.md)

## What is Hexagonal Architecture?

Hexagonal architecture (also known as ports and adapters pattern) creates loosely coupled application components that connect to their software environment through ports and adapters.

### Key Principles

1. **Domain Independence**: Business logic independent of frameworks and external concerns
2. **Testability**: Each layer tested in isolation
3. **Flexibility**: Simple to swap implementations (databases, frameworks, external services)
4. **Clear Boundaries**: Well-defined separation between layers

### Layer Structure

```
┌─────────────────────────────────────────────┐
│         Infrastructure Layer                │
│  (Actix/Axum, Postgres, External APIs)     │
│                                             │
│  ┌───────────────────────────────────────┐ │
│  │      Application Layer                │ │
│  │    (Use Cases, Orchestration)         │ │
│  │                                       │ │
│  │  ┌─────────────────────────────────┐ │ │
│  │  │      Domain Layer               │ │ │
│  │  │  (Entities, Business Logic)     │ │ │
│  │  └─────────────────────────────────┘ │ │
│  └───────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

## Installation

### Personal Skills (Available Across All Projects)

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills

git clone https://github.com/xeros0201/claude-code-skills.git
cp -r claude-code-skills/implementing-hexagonal-actix ./
cp -r claude-code-skills/implementing-hexagonal-axum ./
```

### Project-Specific Skills

```bash
cd your-project
mkdir -p .claude/skills
cd .claude/skills

git clone https://github.com/xeros0201/claude-code-skills.git
cp -r claude-code-skills/implementing-hexagonal-actix ./
cp -r claude-code-skills/implementing-hexagonal-axum ./
```

After installation, restart Claude Code for the skills to be recognized.

## Usage

Invoke skills by describing tasks matching their purpose:

```
"Implement hexagonal architecture with Actix"
"Create a user service using clean architecture with Axum"
"Set up ports and adapters pattern for my Rust API"
"Show me thread-safe repository pattern"
```

Claude Code automatically recognizes when to use these skills based on your request.

## Features

### Progressive Disclosure
- **Quick Start**: Concise SKILL.md files (<250 lines) for fast loading
- **Detailed Guides**: Separate reference files loaded only when needed
- **Efficient Context**: Optimized for Claude's context window

### Comprehensive Coverage
- **Concurrency Patterns**: Arc, Mutex, RwLock, tokio::sync, parking_lot, channels
- **Complete Implementation**: Layer-by-layer guides with full examples
- **Testing Strategies**: Unit, integration, and end-to-end testing
- **Security Best Practices**: Input validation, SQL injection prevention, authentication

### Latest Dependencies
- Flexible version requirements to always use latest compatible versions
- No strict version locks - stays current with ecosystem

## Repository Structure

```
.
├── README.md
├── implementing-hexagonal-actix/
│   ├── SKILL.md              # Quick start (217 lines)
│   ├── CONCURRENCY.md        # Thread-safe patterns
│   ├── IMPLEMENTATION.md     # Complete guide
│   └── TESTING.md            # Testing & security
└── implementing-hexagonal-axum/
    ├── SKILL.md              # Quick start (234 lines)
    ├── CONCURRENCY.md        # Thread-safe patterns
    ├── IMPLEMENTATION.md     # Complete guide
    └── TESTING.md            # Testing & security
```

## Dependencies

Both skills use common Rust dependencies with flexible versions:

**Core:**
- `tokio` - Async runtime
- `async-trait` - Async trait support
- `uuid` - Unique identifiers
- `serde` / `serde_json` - Serialization
- `sqlx` - Type-safe SQL

**Concurrency:**
- `parking_lot` - High-performance synchronization

**Actix-Specific:**
- `actix-web` - Web framework

**Axum-Specific:**
- `axum` - Web framework
- `tower` - Middleware composition
- `tower-http` - HTTP middleware

## Concurrency Patterns Included

- **Arc** - Atomic reference counting for shared ownership
- **Mutex** - Mutual exclusion for exclusive access
- **RwLock** - Read-write locks for read-heavy workloads
- **tokio::sync::Mutex** - Async-aware mutex
- **tokio::sync::RwLock** - Async read-write lock
- **parking_lot** - High-performance synchronization
- **Channels** - Message passing between tasks
- **OnceCell** - Lazy initialization

## Security Features

- Input validation at domain boundaries
- SQL injection prevention with parameterized queries
- Error message sanitization
- CORS configuration
- Rate limiting
- TLS/HTTPS support
- Authentication and authorization patterns
- Request size limits

## Standards Compliance

These skills follow official Claude Code standards:

✅ SKILL.md under 500 lines (217-234 lines)
✅ Progressive disclosure with supporting files
✅ Gerund naming convention
✅ Specific descriptions with trigger terms
✅ Security-first approach
✅ No comments in generated code

## Contributing

Contributions welcome:

- Report issues
- Suggest improvements
- Submit pull requests
- Add skills for other frameworks

## License

MIT

## Related Resources

- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Actix Web Documentation](https://actix.rs/)
- [Axum Documentation](https://docs.rs/axum/)
- [Claude Code Documentation](https://docs.claude.com/)
- [Claude Code Skills Best Practices](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices)
