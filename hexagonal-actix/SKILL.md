---
name: hexagonal-actix
description: Implements hexagonal architecture (ports and adapters pattern) for Rust projects using Actix-web framework. Use when structuring Actix applications with clean separation between domain, application, and infrastructure layers.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Hexagonal Architecture with Actix-web

Implements the hexagonal architecture pattern (also known as ports and adapters) for Rust projects using the Actix-web framework.

## Architecture Overview

```
src/
├── domain/           # Business logic (framework-agnostic)
│   ├── entities/     # Core business objects
│   ├── value_objects/# Immutable domain values
│   ├── services/     # Domain services
│   └── repositories/ # Repository traits (ports)
├── application/      # Use cases and application logic
│   ├── use_cases/    # Application use cases
│   └── ports/        # Input/output ports (traits)
├── infrastructure/   # External concerns (framework-specific)
│   ├── web/          # Actix web handlers and routes
│   ├── persistence/  # Database implementations
│   └── adapters/     # External service adapters
└── main.rs           # Application entry point
```

## Implementation Steps

### 1. Domain Layer

Create domain entities that are framework-agnostic:

```rust
// src/domain/entities/user.rs
use uuid::Uuid;

#[derive(Debug, Clone)]
pub struct User {
    id: Uuid,
    email: String,
    name: String,
}

impl User {
    pub fn new(email: String, name: String) -> Result<Self, String> {
        if email.is_empty() {
            return Err("Email cannot be empty".to_string());
        }
        Ok(Self {
            id: Uuid::new_v4(),
            email,
            name,
        })
    }

    pub fn id(&self) -> &Uuid {
        &self.id
    }

    pub fn email(&self) -> &str {
        &self.email
    }

    pub fn name(&self) -> &str {
        &self.name
    }
}
```

Define repository ports (traits):

```rust
// src/domain/repositories/user_repository.rs
use async_trait::async_trait;
use uuid::Uuid;
use crate::domain::entities::User;

#[async_trait]
pub trait UserRepository: Send + Sync {
    async fn find_by_id(&self, id: &Uuid) -> Result<Option<User>, String>;
    async fn find_by_email(&self, email: &str) -> Result<Option<User>, String>;
    async fn save(&self, user: &User) -> Result<(), String>;
    async fn delete(&self, id: &Uuid) -> Result<(), String>;
}
```

### 2. Application Layer

Define use case input ports:

```rust
// src/application/ports/user_service.rs
use async_trait::async_trait;
use uuid::Uuid;
use crate::domain::entities::User;

#[async_trait]
pub trait UserService: Send + Sync {
    async fn create_user(&self, email: String, name: String) -> Result<User, String>;
    async fn get_user(&self, id: &Uuid) -> Result<Option<User>, String>;
    async fn delete_user(&self, id: &Uuid) -> Result<(), String>;
}
```

Implement use cases:

```rust
// src/application/use_cases/user_use_case.rs
use async_trait::async_trait;
use uuid::Uuid;
use std::sync::Arc;
use crate::domain::entities::User;
use crate::domain::repositories::UserRepository;
use crate::application::ports::UserService;

pub struct UserUseCase {
    repository: Arc<dyn UserRepository>,
}

impl UserUseCase {
    pub fn new(repository: Arc<dyn UserRepository>) -> Self {
        Self { repository }
    }
}

#[async_trait]
impl UserService for UserUseCase {
    async fn create_user(&self, email: String, name: String) -> Result<User, String> {
        if let Some(_) = self.repository.find_by_email(&email).await? {
            return Err("User with this email already exists".to_string());
        }

        let user = User::new(email, name)?;
        self.repository.save(&user).await?;
        Ok(user)
    }

    async fn get_user(&self, id: &Uuid) -> Result<Option<User>, String> {
        self.repository.find_by_id(id).await
    }

    async fn delete_user(&self, id: &Uuid) -> Result<(), String> {
        self.repository.delete(id).await
    }
}
```

### 3. Infrastructure Layer - Persistence Adapter

Implement repository for your chosen database:

```rust
// src/infrastructure/persistence/postgres_user_repository.rs
use async_trait::async_trait;
use uuid::Uuid;
use sqlx::PgPool;
use crate::domain::entities::User;
use crate::domain::repositories::UserRepository;

pub struct PostgresUserRepository {
    pool: PgPool,
}

impl PostgresUserRepository {
    pub fn new(pool: PgPool) -> Self {
        Self { pool }
    }
}

#[async_trait]
impl UserRepository for PostgresUserRepository {
    async fn find_by_id(&self, id: &Uuid) -> Result<Option<User>, String> {
        let result = sqlx::query_as!(
            UserRow,
            "SELECT id, email, name FROM users WHERE id = $1",
            id
        )
        .fetch_optional(&self.pool)
        .await
        .map_err(|e| format!("Database error: {}", e))?;

        Ok(result.map(|row| User::from(row)))
    }

    async fn find_by_email(&self, email: &str) -> Result<Option<User>, String> {
        let result = sqlx::query_as!(
            UserRow,
            "SELECT id, email, name FROM users WHERE email = $1",
            email
        )
        .fetch_optional(&self.pool)
        .await
        .map_err(|e| format!("Database error: {}", e))?;

        Ok(result.map(|row| User::from(row)))
    }

    async fn save(&self, user: &User) -> Result<(), String> {
        sqlx::query!(
            "INSERT INTO users (id, email, name) VALUES ($1, $2, $3)
             ON CONFLICT (id) DO UPDATE SET email = $2, name = $3",
            user.id(),
            user.email(),
            user.name()
        )
        .execute(&self.pool)
        .await
        .map_err(|e| format!("Database error: {}", e))?;

        Ok(())
    }

    async fn delete(&self, id: &Uuid) -> Result<(), String> {
        sqlx::query!("DELETE FROM users WHERE id = $1", id)
            .execute(&self.pool)
            .await
            .map_err(|e| format!("Database error: {}", e))?;

        Ok(())
    }
}

#[derive(sqlx::FromRow)]
struct UserRow {
    id: Uuid,
    email: String,
    name: String,
}
```

### 4. Infrastructure Layer - Actix Web Adapter

Create DTOs for web layer:

```rust
// src/infrastructure/web/dtos/user_dto.rs
use serde::{Deserialize, Serialize};
use uuid::Uuid;
use crate::domain::entities::User;

#[derive(Debug, Serialize)]
pub struct UserResponse {
    pub id: Uuid,
    pub email: String,
    pub name: String,
}

impl From<User> for UserResponse {
    fn from(user: User) -> Self {
        Self {
            id: *user.id(),
            email: user.email().to_string(),
            name: user.name().to_string(),
        }
    }
}

#[derive(Debug, Deserialize)]
pub struct CreateUserRequest {
    pub email: String,
    pub name: String,
}
```

Create Actix handlers:

```rust
// src/infrastructure/web/handlers/user_handler.rs
use actix_web::{web, HttpResponse, Responder};
use uuid::Uuid;
use std::sync::Arc;
use crate::application::ports::UserService;
use crate::infrastructure::web::dtos::user_dto::{CreateUserRequest, UserResponse};

pub async fn create_user(
    service: web::Data<Arc<dyn UserService>>,
    req: web::Json<CreateUserRequest>,
) -> impl Responder {
    match service.create_user(req.email.clone(), req.name.clone()).await {
        Ok(user) => HttpResponse::Created().json(UserResponse::from(user)),
        Err(e) => HttpResponse::BadRequest().json(serde_json::json!({
            "error": e
        })),
    }
}

pub async fn get_user(
    service: web::Data<Arc<dyn UserService>>,
    id: web::Path<Uuid>,
) -> impl Responder {
    match service.get_user(&id).await {
        Ok(Some(user)) => HttpResponse::Ok().json(UserResponse::from(user)),
        Ok(None) => HttpResponse::NotFound().json(serde_json::json!({
            "error": "User not found"
        })),
        Err(e) => HttpResponse::InternalServerError().json(serde_json::json!({
            "error": e
        })),
    }
}

pub async fn delete_user(
    service: web::Data<Arc<dyn UserService>>,
    id: web::Path<Uuid>,
) -> impl Responder {
    match service.delete_user(&id).await {
        Ok(_) => HttpResponse::NoContent().finish(),
        Err(e) => HttpResponse::InternalServerError().json(serde_json::json!({
            "error": e
        })),
    }
}
```

Configure routes:

```rust
// src/infrastructure/web/routes.rs
use actix_web::web;
use crate::infrastructure::web::handlers::user_handler;

pub fn configure_routes(cfg: &mut web::ServiceConfig) {
    cfg.service(
        web::scope("/api/v1")
            .route("/users", web::post().to(user_handler::create_user))
            .route("/users/{id}", web::get().to(user_handler::get_user))
            .route("/users/{id}", web::delete().to(user_handler::delete_user))
    );
}
```

### 5. Application Entry Point

Wire everything together:

```rust
// src/main.rs
use actix_web::{web, App, HttpServer};
use sqlx::postgres::PgPoolOptions;
use std::sync::Arc;

mod domain;
mod application;
mod infrastructure;

use infrastructure::persistence::postgres_user_repository::PostgresUserRepository;
use application::use_cases::user_use_case::UserUseCase;
use infrastructure::web::routes::configure_routes;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let database_url = std::env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");

    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await
        .expect("Failed to create pool");

    let user_repository = Arc::new(PostgresUserRepository::new(pool));
    let user_service: Arc<dyn application::ports::UserService> =
        Arc::new(UserUseCase::new(user_repository));

    HttpServer::new(move || {
        App::new()
            .app_data(web::Data::new(user_service.clone()))
            .configure(configure_routes)
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

## Security Considerations

1. **Input Validation**: Always validate inputs in domain entities
2. **SQL Injection**: Use parameterized queries (sqlx macros)
3. **Error Handling**: Never expose internal errors to clients
4. **Authentication**: Implement in infrastructure layer, enforce in application layer
5. **Authorization**: Handle in application use cases
6. **Rate Limiting**: Configure in Actix middleware
7. **CORS**: Configure appropriately for your use case
8. **TLS**: Always use HTTPS in production

## Required Dependencies

```toml
[dependencies]
actix-web = "4"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
uuid = { version = "1", features = ["v4", "serde"] }
async-trait = "0.1"
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres", "uuid"] }
```

## Testing Strategy

1. **Domain Layer**: Pure unit tests (no mocking needed)
2. **Application Layer**: Mock repositories using traits
3. **Infrastructure Layer**: Integration tests with test database
4. **Web Layer**: Use actix-web test utilities

## Benefits

- Framework independence in business logic
- Easy to test and maintain
- Flexibility to swap implementations
- Clear separation of concerns
- Technology-agnostic domain model
