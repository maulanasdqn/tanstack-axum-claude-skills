---
name: axum-best-practice
description: Enforce Axum/Rust backend best practices when writing Rust code
---

# Axum Backend Best Practices

## Core Principles
- Max 200 LOC per file
- No code comments
- No `unwrap()` in production (use `?` or handle errors)
- Clean Architecture: domain → application → infrastructure
- Single responsibility per file
- Reusability-first design

## Project Structure

```
{prefix}-server/           # Entry point
{prefix}-auth/             # Auth module
{prefix}-users/            # Users module
{prefix}-{feature}/        # Feature modules
{prefix}-database/         # DB connection, entities
{prefix}-errors/           # Centralized AppError
{prefix}-types/            # Shared types (Money, responses)
{prefix}-validation/       # Validation framework
{prefix}-migration/        # DB migrations
```

## Module Structure

Each feature module follows Clean Architecture:

```
{prefix}-{feature}/src/
├── lib.rs
├── domain/
│   ├── mod.rs
│   ├── {entity}.rs        # Entity definition
│   └── repository.rs      # Repository trait
├── application/
│   ├── mod.rs
│   └── {use_case}.rs      # One use case per file
└── infrastructure/
    ├── mod.rs
    ├── http/
    │   ├── handlers.rs
    │   ├── routes.rs
    │   └── dto.rs
    └── persistence/
        └── postgres_{entity}_repository.rs
```

## Error Handling

Centralized `AppError` enum:

```rust
pub enum AppError {
    NotFound(String),
    BadRequest(String),
    ValidationError(String),
    Unauthorized(String),
    Forbidden(String),
    Conflict(String),
    InternalError(String),
}

impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, message) = match self {
            AppError::NotFound(msg) => (StatusCode::NOT_FOUND, msg),
            AppError::BadRequest(msg) => (StatusCode::BAD_REQUEST, msg),
            AppError::Unauthorized(msg) => (StatusCode::UNAUTHORIZED, msg),
            AppError::Forbidden(msg) => (StatusCode::FORBIDDEN, msg),
            AppError::Conflict(msg) => (StatusCode::CONFLICT, msg),
            AppError::InternalError(msg) => (StatusCode::INTERNAL_SERVER_ERROR, msg),
            AppError::ValidationError(msg) => (StatusCode::BAD_REQUEST, msg),
        };
        (status, Json(ErrorResponse { message })).into_response()
    }
}
```

## Repository Pattern

Domain layer defines trait:
```rust
#[async_trait]
pub trait UserRepository: Send + Sync {
    async fn create(&self, user: User) -> Result<User, AppError>;
    async fn find_by_id(&self, id: Uuid) -> Result<Option<User>, AppError>;
    async fn find_by_email(&self, email: &str) -> Result<Option<User>, AppError>;
}
```

Infrastructure implements:
```rust
pub struct PostgresUserRepository {
    db: DatabaseConnection,
}

#[async_trait]
impl UserRepository for PostgresUserRepository {
    async fn create(&self, user: User) -> Result<User, AppError> {
        // Implementation
    }
}
```

## Use Case Pattern

One file per use case:
```rust
pub struct CreateUser {
    user_repo: Arc<dyn UserRepository>,
}

impl CreateUser {
    pub fn new(user_repo: Arc<dyn UserRepository>) -> Self {
        Self { user_repo }
    }

    pub async fn execute(&self, input: CreateUserInput) -> Result<User, AppError> {
        // Business logic
    }
}
```

## Handler Pattern

```rust
pub async fn create_user_handler(
    Extension(use_case): Extension<Arc<CreateUser>>,
    Validated(payload): Validated<CreateUserRequest>,
) -> Result<Json<SingleResponse<UserResponse>>, AppError> {
    let user = use_case.execute(payload.into()).await?;
    Ok(Json(SingleResponse::new(user.into(), "User created")))
}
```

## Response Types

```rust
pub struct SingleResponse<T> {
    pub data: T,
    pub message: String,
}

pub struct ListResponse<T> {
    pub data: Vec<T>,
    pub pagination: PaginationMeta,
}
```

## Rules
1. Domain layer has no external dependencies
2. Repository traits in domain, implementations in infrastructure
3. One use case per file
4. DTOs only in infrastructure/http
5. Use `Arc<dyn Trait>` for dependency injection
6. Handler returns `Result<_, AppError>`
7. Validate input with custom `Validated` extractor
