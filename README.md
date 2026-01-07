# Claude Code Skills - Rust Hexagonal Architecture

A collection of Claude Code skills for implementing hexagonal architecture (ports and adapters pattern) in Rust projects.

## Available Skills

### 1. Hexagonal Architecture with Actix-web
**Skill Name:** `hexagonal-actix`

Implements the hexagonal architecture pattern for Rust projects using the Actix-web framework. This skill helps you structure Actix applications with clean separation between domain, application, and infrastructure layers.

**Use When:**
- Building web services with Actix-web
- Need high performance HTTP server
- Want mature, battle-tested framework
- Building applications with complex routing needs

[View Skill Documentation](./hexagonal-actix/SKILL.md)

### 2. Hexagonal Architecture with Axum
**Skill Name:** `hexagonal-axum`

Implements the hexagonal architecture pattern for Rust projects using the Axum framework. This skill helps you structure Axum applications with clean separation between domain, application, and infrastructure layers.

**Use When:**
- Building web services with Axum
- Want compile-time route validation
- Prefer type-safe extractors and Tower ecosystem
- Need more idiomatic Rust patterns
- Want smaller binary sizes

[View Skill Documentation](./hexagonal-axum/SKILL.md)

## What is Hexagonal Architecture?

Hexagonal architecture (also known as ports and adapters pattern) is a software design pattern that aims to create loosely coupled application components that can be easily connected to their software environment through ports and adapters.

### Key Principles

1. **Domain Independence**: Business logic is independent of frameworks and external concerns
2. **Testability**: Easy to test each layer in isolation
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
cp -r claude-code-skills/hexagonal-actix ./
cp -r claude-code-skills/hexagonal-axum ./
```

### Project-Specific Skills

```bash
cd your-project
mkdir -p .claude/skills
cd .claude/skills

git clone https://github.com/xeros0201/claude-code-skills.git
cp -r claude-code-skills/hexagonal-actix ./
cp -r claude-code-skills/hexagonal-axum ./
```

After installation, restart Claude Code for the skills to be recognized.

## Usage

Once installed, you can invoke the skills by describing tasks that match their purpose:

```
"Help me set up hexagonal architecture with Actix"
"Create a user service using hexagonal architecture with Axum"
"Implement a repository pattern for my Actix application"
```

Claude Code will automatically recognize when to use these skills based on your request.

## Common Use Cases

### Starting a New Project
1. Choose your framework (Actix or Axum)
2. Ask Claude to help structure your project using hexagonal architecture
3. Follow the generated structure for domain, application, and infrastructure layers

### Refactoring Existing Code
1. Identify your business logic (domain layer)
2. Extract use cases (application layer)
3. Move framework-specific code to infrastructure layer

### Adding New Features
1. Define domain entities and business rules
2. Create use cases in application layer
3. Implement infrastructure adapters (HTTP handlers, repositories)

## Benefits of Using These Skills

- **Consistency**: Follow established patterns across all your Rust projects
- **Best Practices**: Security-first approach with built-in validation and error handling
- **Productivity**: Quick scaffolding of clean architecture structures
- **Maintainability**: Clear separation of concerns makes code easier to maintain
- **Testability**: Each layer can be tested independently

## Repository Structure

```
.
├── README.md
├── hexagonal-actix/
│   └── SKILL.md
└── hexagonal-axum/
    └── SKILL.md
```

## Dependencies

Both skills use common Rust dependencies:

- **async-trait**: For async trait support
- **uuid**: For unique identifiers
- **serde**: For serialization/deserialization
- **sqlx**: For database operations (type-safe SQL)
- **tokio**: Async runtime

### Actix-Specific
- **actix-web**: Web framework

### Axum-Specific
- **axum**: Web framework
- **tower**: Middleware and service composition
- **tower-http**: HTTP-specific middleware

## Security Features

Both skills implement security best practices:

- Input validation at domain boundaries
- SQL injection prevention using parameterized queries
- Error message sanitization
- CORS configuration
- Rate limiting guidance
- TLS/HTTPS support
- Authentication and authorization patterns

## Contributing

Contributions are welcome! Feel free to:

- Report issues
- Suggest improvements
- Submit pull requests
- Add new skills for other frameworks

## License

MIT

## Author

Created for use with Claude Code by Anthropic.

## Related Resources

- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [Actix Web Documentation](https://actix.rs/)
- [Axum Documentation](https://docs.rs/axum/)
- [Claude Code Documentation](https://docs.claude.com/)
