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

## Rust Concurrency Patterns

### Arc - Atomic Reference Counting

Use `Arc` for shared ownership across threads. Essential for sharing services and repositories.

```rust
use std::sync::Arc;

let repository = Arc::new(PostgresUserRepository::new(pool));
let service: Arc<dyn UserService> = Arc::new(UserUseCase::new(repository.clone()));

let service_clone = service.clone();
tokio::spawn(async move {
    service_clone.get_user(&user_id).await
});
```

### Mutex - Mutual Exclusion

Use `Mutex` for shared mutable state with exclusive access. Blocks threads waiting for lock.

```rust
use std::sync::{Arc, Mutex};

pub struct CachedUserRepository {
    inner: Arc<dyn UserRepository>,
    cache: Arc<Mutex<HashMap<Uuid, User>>>,
}

impl CachedUserRepository {
    pub fn new(inner: Arc<dyn UserRepository>) -> Self {
        Self {
            inner,
            cache: Arc::new(Mutex::new(HashMap::new())),
        }
    }
}

#[async_trait]
impl UserRepository for CachedUserRepository {
    async fn find_by_id(&self, id: &Uuid) -> Result<Option<User>, String> {
        {
            let cache = self.cache.lock().unwrap();
            if let Some(user) = cache.get(id) {
                return Ok(Some(user.clone()));
            }
        }

        let user = self.inner.find_by_id(id).await?;
        if let Some(ref u) = user {
            let mut cache = self.cache.lock().unwrap();
            cache.insert(*id, u.clone());
        }
        Ok(user)
    }

    async fn save(&self, user: &User) -> Result<(), String> {
        self.inner.save(user).await?;
        let mut cache = self.cache.lock().unwrap();
        cache.insert(*user.id(), user.clone());
        Ok(())
    }

    async fn delete(&self, id: &Uuid) -> Result<(), String> {
        self.inner.delete(id).await?;
        let mut cache = self.cache.lock().unwrap();
        cache.remove(id);
        Ok(())
    }

    async fn find_by_email(&self, email: &str) -> Result<Option<User>, String> {
        self.inner.find_by_email(email).await
    }
}
```

### RwLock - Read-Write Lock

Use `RwLock` for shared state with multiple readers or single writer. Better performance than Mutex for read-heavy workloads.

```rust
use std::sync::{Arc, RwLock};
use std::collections::HashMap;

pub struct ConfigurationService {
    settings: Arc<RwLock<HashMap<String, String>>>,
}

impl ConfigurationService {
    pub fn new() -> Self {
        Self {
            settings: Arc::new(RwLock::new(HashMap::new())),
        }
    }

    pub fn get(&self, key: &str) -> Option<String> {
        let settings = self.settings.read().unwrap();
        settings.get(key).cloned()
    }

    pub fn set(&self, key: String, value: String) {
        let mut settings = self.settings.write().unwrap();
        settings.insert(key, value);
    }

    pub fn get_all(&self) -> HashMap<String, String> {
        let settings = self.settings.read().unwrap();
        settings.clone()
    }
}
```

### Tokio Mutex - Async-Aware Mutex

Use `tokio::sync::Mutex` for async contexts. Yields instead of blocking threads.

```rust
use tokio::sync::Mutex;
use std::sync::Arc;

pub struct AsyncCachedRepository {
    inner: Arc<dyn UserRepository>,
    cache: Arc<Mutex<HashMap<Uuid, User>>>,
}

#[async_trait]
impl UserRepository for AsyncCachedRepository {
    async fn find_by_id(&self, id: &Uuid) -> Result<Option<User>, String> {
        {
            let cache = self.cache.lock().await;
            if let Some(user) = cache.get(id) {
                return Ok(Some(user.clone()));
            }
        }

        let user = self.inner.find_by_id(id).await?;
        if let Some(ref u) = user {
            let mut cache = self.cache.lock().await;
            cache.insert(*id, u.clone());
        }
        Ok(user)
    }

    async fn save(&self, user: &User) -> Result<(), String> {
        self.inner.save(user).await?;
        let mut cache = self.cache.lock().await;
        cache.insert(*user.id(), user.clone());
        Ok(())
    }

    async fn delete(&self, id: &Uuid) -> Result<(), String> {
        self.inner.delete(id).await?;
        let mut cache = self.cache.lock().await;
        cache.remove(id);
        Ok(())
    }

    async fn find_by_email(&self, email: &str) -> Result<Option<User>, String> {
        self.inner.find_by_email(email).await
    }
}
```

### Tokio RwLock - Async Read-Write Lock

Use `tokio::sync::RwLock` for async read-write scenarios.

```rust
use tokio::sync::RwLock;
use std::sync::Arc;

pub struct AsyncConfigService {
    settings: Arc<RwLock<HashMap<String, String>>>,
}

impl AsyncConfigService {
    pub fn new() -> Self {
        Self {
            settings: Arc::new(RwLock::new(HashMap::new())),
        }
    }

    pub async fn get(&self, key: &str) -> Option<String> {
        let settings = self.settings.read().await;
        settings.get(key).cloned()
    }

    pub async fn set(&self, key: String, value: String) {
        let mut settings = self.settings.write().await;
        settings.insert(key, value);
    }
}
```

### Parking Lot - High-Performance Synchronization

Use `parking_lot` for better performance than standard library primitives.

```rust
use parking_lot::{Mutex, RwLock};
use std::sync::Arc;

pub struct FastCachedRepository {
    inner: Arc<dyn UserRepository>,
    cache: Arc<Mutex<HashMap<Uuid, User>>>,
}

#[async_trait]
impl UserRepository for FastCachedRepository {
    async fn find_by_id(&self, id: &Uuid) -> Result<Option<User>, String> {
        {
            let cache = self.cache.lock();
            if let Some(user) = cache.get(id) {
                return Ok(Some(user.clone()));
            }
        }

        let user = self.inner.find_by_id(id).await?;
        if let Some(ref u) = user {
            self.cache.lock().insert(*id, u.clone());
        }
        Ok(user)
    }

    async fn save(&self, user: &User) -> Result<(), String> {
        self.inner.save(user).await?;
        self.cache.lock().insert(*user.id(), user.clone());
        Ok(())
    }

    async fn delete(&self, id: &Uuid) -> Result<(), String> {
        self.inner.delete(id).await?;
        self.cache.lock().remove(id);
        Ok(())
    }

    async fn find_by_email(&self, email: &str) -> Result<Option<User>, String> {
        self.inner.find_by_email(email).await
    }
}
```

### Channels - Message Passing

Use channels for communication between tasks.

```rust
use tokio::sync::mpsc;

pub struct EventPublisher {
    tx: mpsc::UnboundedSender<DomainEvent>,
}

pub enum DomainEvent {
    UserCreated(Uuid),
    UserDeleted(Uuid),
}

impl EventPublisher {
    pub fn new() -> (Self, mpsc::UnboundedReceiver<DomainEvent>) {
        let (tx, rx) = mpsc::unbounded_channel();
        (Self { tx }, rx)
    }

    pub fn publish(&self, event: DomainEvent) {
        let _ = self.tx.send(event);
    }
}

pub struct UserUseCaseWithEvents {
    repository: Arc<dyn UserRepository>,
    publisher: Arc<EventPublisher>,
}

#[async_trait]
impl UserService for UserUseCaseWithEvents {
    async fn create_user(&self, email: String, name: String) -> Result<User, String> {
        let user = User::new(email, name)?;
        self.repository.save(&user).await?;
        self.publisher.publish(DomainEvent::UserCreated(*user.id()));
        Ok(user)
    }

    async fn delete_user(&self, id: &Uuid) -> Result<(), String> {
        self.repository.delete(id).await?;
        self.publisher.publish(DomainEvent::UserDeleted(*id));
        Ok(())
    }

    async fn get_user(&self, id: &Uuid) -> Result<Option<User>, String> {
        self.repository.find_by_id(id).await
    }
}
```

### OnceCell - Lazy Initialization

Use `OnceCell` for one-time initialization.

```rust
use std::sync::OnceLock;

static CONFIG: OnceLock<AppConfig> = OnceLock::new();

pub struct AppConfig {
    pub database_url: String,
    pub port: u16,
}

pub fn get_config() -> &'static AppConfig {
    CONFIG.get_or_init(|| AppConfig {
        database_url: std::env::var("DATABASE_URL").unwrap(),
        port: 8080,
    })
}
```

### Concurrency Best Practices

1. **Prefer Arc over cloning**: Share ownership instead of cloning expensive data
2. **Use RwLock for read-heavy**: Multiple readers can access simultaneously
3. **Avoid holding locks across await**: Deadlock risk and performance issues
4. **Use parking_lot**: Better performance than std primitives
5. **Prefer message passing**: Channels over shared state when possible
6. **Tokio primitives for async**: Use tokio::sync for async contexts
7. **Minimize lock scope**: Release locks as soon as possible
8. **Avoid nested locks**: Prevent deadlocks

### Thread-Safe Repository Pattern

Complete example with proper concurrency:

```rust
use std::sync::Arc;
use parking_lot::RwLock;
use std::collections::HashMap;

pub struct InMemoryUserRepository {
    users: Arc<RwLock<HashMap<Uuid, User>>>,
    email_index: Arc<RwLock<HashMap<String, Uuid>>>,
}

impl InMemoryUserRepository {
    pub fn new() -> Self {
        Self {
            users: Arc::new(RwLock::new(HashMap::new())),
            email_index: Arc::new(RwLock::new(HashMap::new())),
        }
    }
}

#[async_trait]
impl UserRepository for InMemoryUserRepository {
    async fn find_by_id(&self, id: &Uuid) -> Result<Option<User>, String> {
        let users = self.users.read();
        Ok(users.get(id).cloned())
    }

    async fn find_by_email(&self, email: &str) -> Result<Option<User>, String> {
        let email_index = self.email_index.read();
        let user_id = email_index.get(email);

        match user_id {
            Some(id) => {
                let users = self.users.read();
                Ok(users.get(id).cloned())
            }
            None => Ok(None),
        }
    }

    async fn save(&self, user: &User) -> Result<(), String> {
        let mut users = self.users.write();
        let mut email_index = self.email_index.write();

        users.insert(*user.id(), user.clone());
        email_index.insert(user.email().to_string(), *user.id());

        Ok(())
    }

    async fn delete(&self, id: &Uuid) -> Result<(), String> {
        let mut users = self.users.write();
        let mut email_index = self.email_index.write();

        if let Some(user) = users.remove(id) {
            email_index.remove(user.email());
        }

        Ok(())
    }
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
9. **Concurrency Safety**: Use proper synchronization primitives
10. **Lock Poisoning**: Handle or prevent poisoned locks

## Required Dependencies

```toml
[dependencies]
actix-web = "4"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
uuid = { version = "1", features = ["v4", "serde"] }
async-trait = "0.1"
sqlx = { version = "0", features = ["runtime-tokio-rustls", "postgres", "uuid"] }
parking_lot = "0"
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
