# Claude Code Skills

Professional Claude Code skills for modern development. Includes hexagonal architecture for Rust web services, Svelte 5 / SvelteKit for frontend development, and Tauri for cross-platform desktop applications. Optimized for performance with progressive disclosure and comprehensive guides.

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

### 3. Svelte 5 and SvelteKit Development
**Skill Name:** `svelte5-sveltekit`

Guides development of modern web applications with Svelte 5 and SvelteKit. Features progressive disclosure with concise quick-start and detailed reference files.

**Use When:**
- Building web applications with Svelte 5
- Creating SvelteKit routes and layouts
- Implementing data loading and form actions
- Building API endpoints
- Using Svelte 5 runes ($state, $derived, $effect)
- Working with snippets and component composition

**Structure:**
- `SKILL.md` - Quick start and overview
- `COMPONENTS.md` - Deep dive on Svelte 5 runes, snippets, and patterns
- `SVELTEKIT.md` - Complete guide to routing, data loading, hooks
- `TESTING.md` - Testing strategies and security practices

[View Skill](./svelte5-sveltekit/SKILL.md)

### 4. Tauri Desktop Application Development
**Skill Name:** `tauri-desktop`

Guides development of cross-platform desktop applications with Tauri using Rust backend and web frontend. Features progressive disclosure with comprehensive patterns and security-first approach.

**Use When:**
- Building cross-platform desktop applications
- Creating apps with Rust backend and web UI
- Implementing secure IPC between frontend and backend
- Managing application windows and system tray
- Accessing file system and system APIs
- Building distributable desktop apps

**Structure:**
- `SKILL.md` - Quick start with commands, state, events, and database
- `ARCHITECTURE.md` - Clean architecture patterns, state management, error handling
- `FRONTEND-INTEGRATION.md` - React/Vue/Svelte integration, TypeScript bindings, IPC patterns
- `TESTING.md` - Unit, integration, E2E testing, and security practices

[View Skill](./tauri-desktop/SKILL.md)

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
cp -r claude-code-skills/svelte5-sveltekit ./
cp -r claude-code-skills/tauri-desktop ./
```

### Project-Specific Skills

```bash
cd your-project
mkdir -p .claude/skills
cd .claude/skills

git clone https://github.com/xeros0201/claude-code-skills.git
cp -r claude-code-skills/implementing-hexagonal-actix ./
cp -r claude-code-skills/implementing-hexagonal-axum ./
cp -r claude-code-skills/svelte5-sveltekit ./
cp -r claude-code-skills/tauri-desktop ./
```

After installation, restart Claude Code for the skills to be recognized.

## Usage

Invoke skills by describing tasks matching their purpose:

**Rust Hexagonal Architecture:**
```
"Implement hexagonal architecture with Actix"
"Create a user service using clean architecture with Axum"
"Set up ports and adapters pattern for my Rust API"
"Show me thread-safe repository pattern"
```

**Svelte 5 / SvelteKit:**
```
"Create a Svelte 5 component with state management"
"Build a SvelteKit route with data loading"
"Implement form actions with validation"
"Set up API endpoints in SvelteKit"
```

**Tauri Desktop:**
```
"Build a Tauri desktop app with file management"
"Create Tauri commands for database operations"
"Implement window management in Tauri"
"Set up secure IPC between frontend and Rust backend"
```

Claude Code automatically recognizes when to use these skills based on your request.

## Features

### Progressive Disclosure
- **Quick Start**: Concise SKILL.md files (<500 lines) for fast loading
- **Detailed Guides**: Separate reference files loaded only when needed
- **Efficient Context**: Optimized for Claude's context window

### Comprehensive Coverage

**Rust Skills:**
- **Concurrency Patterns**: Arc, Mutex, RwLock, tokio::sync, parking_lot, channels
- **Complete Implementation**: Layer-by-layer guides with full examples
- **Testing Strategies**: Unit, integration, and end-to-end testing
- **Security Best Practices**: Input validation, SQL injection prevention, authentication

**Svelte 5 / SvelteKit:**
- **Modern Patterns**: Svelte 5 runes, snippets, reactive state management
- **Full-Stack Development**: Routing, data loading, form actions, API endpoints
- **Type Safety**: TypeScript-first with full type inference
- **Testing**: Vitest for unit/component tests, Playwright for E2E
- **Security**: XSS prevention, CSRF protection, secure authentication

**Tauri Desktop:**
- **Cross-Platform**: Windows, macOS, Linux from single codebase
- **Clean Architecture**: Service layer, repository pattern, dependency injection
- **Type-Safe IPC**: Auto-generated TypeScript bindings with specta
- **Frontend Agnostic**: React, Vue, Svelte integration patterns
- **Security First**: Capability-based permissions, CSP, input validation
- **Small Bundle**: ~600KB vs Electron's ~50MB

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
├── implementing-hexagonal-axum/
│   ├── SKILL.md              # Quick start (234 lines)
│   ├── CONCURRENCY.md        # Thread-safe patterns
│   ├── IMPLEMENTATION.md     # Complete guide
│   └── TESTING.md            # Testing & security
├── svelte5-sveltekit/
│   ├── SKILL.md              # Quick start and overview
│   ├── COMPONENTS.md         # Svelte 5 runes and snippets
│   ├── SVELTEKIT.md          # Routing and data loading
│   └── TESTING.md            # Testing & security
└── tauri-desktop/
    ├── SKILL.md              # Quick start with Tauri patterns
    ├── ARCHITECTURE.md       # Clean architecture and state management
    ├── FRONTEND-INTEGRATION.md  # React/Vue/Svelte integration
    └── TESTING.md            # Testing & security
```

## Dependencies

### Rust Skills

Common Rust dependencies with flexible versions:

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

### Svelte 5 / SvelteKit

**Core:**
- `svelte` (v5+) - Reactive UI framework
- `@sveltejs/kit` (v2+) - Full-stack framework
- `@sveltejs/vite-plugin-svelte` - Vite integration

**Development:**
- `vite` - Build tool
- `typescript` - Type safety
- `vitest` - Unit testing
- `@playwright/test` - E2E testing
- `@testing-library/svelte` - Component testing

**Adapters (choose based on deployment):**
- `@sveltejs/adapter-auto` - Auto-detect platform
- `@sveltejs/adapter-node` - Node.js
- `@sveltejs/adapter-static` - Static site generation
- `@sveltejs/adapter-vercel` - Vercel
- `@sveltejs/adapter-cloudflare` - Cloudflare

### Tauri Desktop

**Core:**
- `tauri` (v2+) - Desktop application framework
- `serde` / `serde_json` - Serialization
- `tokio` - Async runtime
- `sqlx` - Database (optional)
- `thiserror` - Error handling

**Type Safety:**
- `tauri-specta` - Auto-generate TypeScript bindings
- `specta` - Type reflection

**Security:**
- `validator` - Input validation
- `keyring` - Secure credential storage

**Frontend (in ui/):**
- React / Vue / Svelte - Choose your framework
- `@tauri-apps/api` - Tauri JavaScript/TypeScript API
- TypeScript for type safety

## Key Patterns Included

### Rust Concurrency Patterns

- **Arc** - Atomic reference counting for shared ownership
- **Mutex** - Mutual exclusion for exclusive access
- **RwLock** - Read-write locks for read-heavy workloads
- **tokio::sync::Mutex** - Async-aware mutex
- **tokio::sync::RwLock** - Async read-write lock
- **parking_lot** - High-performance synchronization
- **Channels** - Message passing between tasks
- **OnceCell** - Lazy initialization

### Svelte 5 Reactive Patterns

- **$state** - Fine-grained reactive state
- **$derived** - Computed values with automatic dependency tracking
- **$effect** - Side effects with automatic cleanup
- **$props** - Type-safe component props
- **$bindable** - Two-way binding for components
- **Snippets** - Reusable markup templates

## Security Features

### Rust Skills

- Input validation at domain boundaries
- SQL injection prevention with parameterized queries
- Error message sanitization
- CORS configuration
- Rate limiting
- TLS/HTTPS support
- Authentication and authorization patterns
- Request size limits

### Svelte 5 / SvelteKit

- XSS prevention with HTML sanitization
- CSRF protection (built-in for form actions)
- Secure cookie handling
- Content Security Policy headers
- Input validation on server and client
- Rate limiting middleware
- Secure password hashing (argon2)
- Environment variable protection

### Tauri Desktop

- Capability-based permission system
- Content Security Policy (CSP)
- Path traversal prevention
- SQL injection prevention with parameterized queries
- Secure credential storage with keyring
- Input validation at command boundaries
- Rate limiting for commands
- Sandboxing and process isolation

## Standards Compliance

All skills follow official Claude Code standards:

✅ SKILL.md under 500 lines
✅ Progressive disclosure with supporting files
✅ Gerund naming convention (implementing-*, svelte5-sveltekit)
✅ Specific descriptions with trigger terms
✅ Security-first approach
✅ No comments in generated code
✅ Type safety and modern patterns

## Contributing

Contributions welcome:

- Report issues
- Suggest improvements
- Submit pull requests
- Add skills for other frameworks

## License

MIT

## Related Resources

**Rust:**
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Actix Web Documentation](https://actix.rs/)
- [Axum Documentation](https://docs.rs/axum/)

**Svelte 5 / SvelteKit:**
- [Svelte 5 Documentation](https://svelte.dev/docs/svelte/overview)
- [SvelteKit Documentation](https://svelte.dev/docs/kit/introduction)
- [Svelte 5 Runes](https://svelte.dev/docs/svelte/runes)
- [SvelteKit Tutorial](https://svelte.dev/tutorial/kit/introducing-sveltekit)

**Tauri:**
- [Tauri Documentation](https://tauri.app/)
- [Tauri API Reference](https://tauri.app/v2/reference/)
- [Tauri Guides](https://tauri.app/v2/guides/)
- [Tauri Examples](https://github.com/tauri-apps/tauri/tree/dev/examples)

**Claude Code:**
- [Claude Code Documentation](https://docs.claude.com/)
- [Claude Code Skills Best Practices](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices)
