---
name: hexagonal-axum
description: Implements hexagonal architecture (ports and adapters pattern) for Rust projects using Axum framework. Use when structuring Axum applications with clean separation between domain, application, and infrastructure layers.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Hexagonal Architecture with Axum

Implements the hexagonal architecture pattern (also known as ports and adapters) for Rust projects using the Axum framework.

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
│   ├── web/          # Axum handlers and routes
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

### 4. Infrastructure Layer - Axum Web Adapter

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

#[derive(Debug, Serialize)]
pub struct ErrorResponse {
    pub error: String,
}
```

Create Axum handlers:

```rust
// src/infrastructure/web/handlers/user_handler.rs
use axum::{
    extract::{Path, State},
    http::StatusCode,
    Json,
    response::IntoResponse,
};
use uuid::Uuid;
use std::sync::Arc;
use crate::application::ports::UserService;
use crate::infrastructure::web::dtos::user_dto::{CreateUserRequest, UserResponse, ErrorResponse};

pub async fn create_user(
    State(service): State<Arc<dyn UserService>>,
    Json(req): Json<CreateUserRequest>,
) -> impl IntoResponse {
    match service.create_user(req.email, req.name).await {
        Ok(user) => (StatusCode::CREATED, Json(UserResponse::from(user))).into_response(),
        Err(e) => (
            StatusCode::BAD_REQUEST,
            Json(ErrorResponse { error: e }),
        ).into_response(),
    }
}

pub async fn get_user(
    State(service): State<Arc<dyn UserService>>,
    Path(id): Path<Uuid>,
) -> impl IntoResponse {
    match service.get_user(&id).await {
        Ok(Some(user)) => (StatusCode::OK, Json(UserResponse::from(user))).into_response(),
        Ok(None) => (
            StatusCode::NOT_FOUND,
            Json(ErrorResponse {
                error: "User not found".to_string(),
            }),
        ).into_response(),
        Err(e) => (
            StatusCode::INTERNAL_SERVER_ERROR,
            Json(ErrorResponse { error: e }),
        ).into_response(),
    }
}

pub async fn delete_user(
    State(service): State<Arc<dyn UserService>>,
    Path(id): Path<Uuid>,
) -> impl IntoResponse {
    match service.delete_user(&id).await {
        Ok(_) => StatusCode::NO_CONTENT.into_response(),
        Err(e) => (
            StatusCode::INTERNAL_SERVER_ERROR,
            Json(ErrorResponse { error: e }),
        ).into_response(),
    }
}
```

Configure routes:

```rust
// src/infrastructure/web/routes.rs
use axum::{
    routing::{get, post, delete},
    Router,
};
use std::sync::Arc;
use crate::application::ports::UserService;
use crate::infrastructure::web::handlers::user_handler;

pub fn create_router(user_service: Arc<dyn UserService>) -> Router {
    Router::new()
        .route("/api/v1/users", post(user_handler::create_user))
        .route("/api/v1/users/:id", get(user_handler::get_user))
        .route("/api/v1/users/:id", delete(user_handler::delete_user))
        .with_state(user_service)
}
```

### 5. Application Entry Point

Wire everything together:

```rust
// src/main.rs
use sqlx::postgres::PgPoolOptions;
use std::sync::Arc;
use tokio::net::TcpListener;

mod domain;
mod application;
mod infrastructure;

use infrastructure::persistence::postgres_user_repository::PostgresUserRepository;
use application::use_cases::user_use_case::UserUseCase;
use infrastructure::web::routes::create_router;

#[tokio::main]
async fn main() {
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

    let app = create_router(user_service);

    let listener = TcpListener::bind("127.0.0.1:8080")
        .await
        .expect("Failed to bind");

    axum::serve(listener, app)
        .await
        .expect("Failed to start server");
}
```

### 6. Application State Pattern (Alternative)

For more complex applications, use a shared state struct:

```rust
// src/infrastructure/web/app_state.rs
use std::sync::Arc;
use crate::application::ports::UserService;

#[derive(Clone)]
pub struct AppState {
    pub user_service: Arc<dyn UserService>,
}

impl AppState {
    pub fn new(user_service: Arc<dyn UserService>) -> Self {
        Self { user_service }
    }
}
```

Update handlers to use AppState:

```rust
pub async fn create_user(
    State(state): State<AppState>,
    Json(req): Json<CreateUserRequest>,
) -> impl IntoResponse {
    match state.user_service.create_user(req.email, req.name).await {
        Ok(user) => (StatusCode::CREATED, Json(UserResponse::from(user))).into_response(),
        Err(e) => (
            StatusCode::BAD_REQUEST,
            Json(ErrorResponse { error: e }),
        ).into_response(),
    }
}
```

### 7. Middleware for Cross-Cutting Concerns

Add logging and CORS:

```rust
// src/infrastructure/web/middleware.rs
use axum::{
    middleware::{self, Next},
    response::Response,
    http::Request,
};
use tower_http::cors::{CorsLayer, Any};

pub async fn logging_middleware<B>(
    req: Request<B>,
    next: Next<B>,
) -> Response {
    let method = req.method().clone();
    let uri = req.uri().clone();

    let response = next.run(req).await;

    println!("{} {} - {}", method, uri, response.status());

    response
}

pub fn cors_layer() -> CorsLayer {
    CorsLayer::new()
        .allow_origin(Any)
        .allow_methods(Any)
        .allow_headers(Any)
}
```

Apply middleware in router:

```rust
use tower_http::cors::CorsLayer;
use axum::middleware;

pub fn create_router(user_service: Arc<dyn UserService>) -> Router {
    Router::new()
        .route("/api/v1/users", post(user_handler::create_user))
        .route("/api/v1/users/:id", get(user_handler::get_user))
        .route("/api/v1/users/:id", delete(user_handler::delete_user))
        .layer(middleware::from_fn(logging_middleware))
        .layer(cors_layer())
        .with_state(user_service)
}
```

## Security Considerations

1. **Input Validation**: Always validate inputs in domain entities
2. **SQL Injection**: Use parameterized queries (sqlx macros)
3. **Error Handling**: Never expose internal errors to clients
4. **Authentication**: Implement using tower middleware
5. **Authorization**: Handle in application use cases
6. **Rate Limiting**: Use tower-governor or similar
7. **CORS**: Configure appropriately for your use case
8. **TLS**: Always use HTTPS in production with rustls
9. **Request Size Limits**: Configure in Axum

## Required Dependencies

```toml
[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
uuid = { version = "1", features = ["v4", "serde"] }
async-trait = "0.1"
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres", "uuid"] }
tower = "0.4"
tower-http = { version = "0.5", features = ["cors", "trace"] }
```

## Testing Strategy

1. **Domain Layer**: Pure unit tests (no mocking needed)
2. **Application Layer**: Mock repositories using traits
3. **Infrastructure Layer**: Integration tests with test database
4. **Web Layer**: Use axum test utilities with tower::ServiceExt

Example handler test:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use axum::body::Body;
    use axum::http::{Request, StatusCode};
    use tower::ServiceExt;

    #[tokio::test]
    async fn test_create_user() {
        let app = create_router(mock_user_service());

        let response = app
            .oneshot(
                Request::builder()
                    .method("POST")
                    .uri("/api/v1/users")
                    .header("content-type", "application/json")
                    .body(Body::from(r#"{"email":"test@example.com","name":"Test"}"#))
                    .unwrap(),
            )
            .await
            .unwrap();

        assert_eq!(response.status(), StatusCode::CREATED);
    }
}
```

## Benefits

- Framework independence in business logic
- Easy to test and maintain
- Flexibility to swap implementations
- Clear separation of concerns
- Technology-agnostic domain model
- Type-safe routing with Axum extractors
- Composable middleware with Tower

## Axum-Specific Advantages

- Compile-time route validation
- Type-safe extractors
- Built on Tower for composable middleware
- Better ergonomics with async/await
- Smaller binary size compared to Actix
- More idiomatic Rust patterns
