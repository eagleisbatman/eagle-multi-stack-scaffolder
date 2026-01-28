# Rust Backend Reference

## Research Queries
- "Rust web framework comparison 2025 Axum Actix"
- "Rust backend best practices 2025"
- "Rust async runtime Tokio"
- "Rust ORM SQLx Diesel SeaORM 2025"

## Package Manager
**Cargo** - Standard Rust package manager.

```bash
cargo new my-api --name my_api
cd my-api
```

## Framework: Axum (Recommended)

Tokio-native, tower ecosystem, modern design.

```toml
# Cargo.toml
[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
tower = "0.4"
tower-http = { version = "0.5", features = ["cors", "trace"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
sqlx = { version = "0.7", features = ["runtime-tokio", "postgres", "uuid"] }
thiserror = "1"
anyhow = "1"
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
uuid = { version = "1", features = ["v4", "serde"] }
```

## Project Structure

```
src/
├── main.rs
├── lib.rs
├── config/
│   ├── mod.rs
│   └── settings.rs
├── api/
│   ├── mod.rs
│   ├── routes.rs
│   ├── handlers/
│   │   ├── mod.rs
│   │   └── farmers.rs
│   └── middleware/
├── domain/
│   ├── mod.rs
│   ├── models/
│   ├── services/
│   └── repositories/
├── infrastructure/
│   ├── mod.rs
│   └── database/
└── error.rs
```

## Code Patterns

### Router
```rust
use axum::{routing::{get, post}, Router};
use sqlx::PgPool;
use tower_http::{cors::CorsLayer, trace::TraceLayer};

pub fn create_router(pool: PgPool) -> Router {
    Router::new()
        .route("/health", get(|| async { "ok" }))
        .route("/farmers", get(list_farmers).post(create_farmer))
        .route("/farmers/:id", get(get_farmer))
        .with_state(pool)
        .layer(TraceLayer::new_for_http())
        .layer(CorsLayer::permissive())
}
```

### Handler
```rust
use axum::{extract::{Path, State}, Json};
use sqlx::PgPool;

pub async fn list_farmers(
    State(pool): State<PgPool>,
) -> Result<Json<Vec<Farmer>>, AppError> {
    let farmers = sqlx::query_as!(Farmer, "SELECT * FROM farmers")
        .fetch_all(&pool)
        .await?;
    Ok(Json(farmers))
}

pub async fn get_farmer(
    State(pool): State<PgPool>,
    Path(id): Path<Uuid>,
) -> Result<Json<Farmer>, AppError> {
    let farmer = sqlx::query_as!(Farmer, "SELECT * FROM farmers WHERE id = $1", id)
        .fetch_optional(&pool)
        .await?
        .ok_or(AppError::NotFound)?;
    Ok(Json(farmer))
}
```

### Error Handling
```rust
use axum::{http::StatusCode, response::{IntoResponse, Response}, Json};
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Not found")]
    NotFound,
    #[error("Database error")]
    Database(#[from] sqlx::Error),
    #[error("Internal error")]
    Internal(#[from] anyhow::Error),
}

impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let status = match &self {
            AppError::NotFound => StatusCode::NOT_FOUND,
            _ => StatusCode::INTERNAL_SERVER_ERROR,
        };
        (status, Json(serde_json::json!({"error": self.to_string()}))).into_response()
    }
}
```

### Main
```rust
#[tokio::main]
async fn main() -> anyhow::Result<()> {
    tracing_subscriber::init();
    
    let pool = PgPool::connect(&std::env::var("DATABASE_URL")?).await?;
    sqlx::migrate!().run(&pool).await?;
    
    let app = create_router(pool);
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await?;
    axum::serve(listener, app).await?;
    Ok(())
}
```

## Setup Commands

```bash
cargo new my-api
cd my-api

# Edit Cargo.toml with dependencies

# Database
export DATABASE_URL="postgresql://user:pass@localhost/db"
cargo install sqlx-cli
sqlx database create
sqlx migrate add initial
sqlx migrate run

# Run
cargo run

# Watch mode
cargo install cargo-watch
cargo watch -x run
```
