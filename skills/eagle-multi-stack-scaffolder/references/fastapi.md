# FastAPI Reference

## Research Queries
- "FastAPI best practices 2025 2026"
- "FastAPI project structure large applications"
- "FastAPI vs Django vs Flask 2025"
- "FastAPI SQLAlchemy vs SQLModel 2025"
- "FastAPI authentication JWT OAuth 2025"

## Package Manager
**uv** - 10-100x faster than pip, proper dependency resolution, built-in virtual environment management.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv init my-api && cd my-api
uv add fastapi uvicorn[standard]
```

## Project Structure

```
src/
├── main.py                      # Application entry point
├── config/
│   ├── __init__.py
│   ├── settings.py              # Pydantic settings
│   └── database.py              # Database connection
├── api/
│   ├── __init__.py
│   ├── deps.py                  # Shared dependencies
│   └── v1/
│       ├── __init__.py
│       ├── router.py            # API router aggregator
│       └── endpoints/
│           ├── __init__.py
│           ├── users.py
│           ├── auth.py
│           └── farmers.py
├── core/
│   ├── __init__.py
│   ├── security.py              # JWT, hashing
│   └── exceptions.py            # Custom exceptions
├── models/
│   ├── __init__.py
│   ├── base.py                  # Base model class
│   ├── user.py
│   └── farmer.py
├── schemas/
│   ├── __init__.py
│   ├── user.py                  # Pydantic schemas
│   └── farmer.py
├── services/
│   ├── __init__.py
│   ├── user_service.py
│   └── farmer_service.py
├── repositories/
│   ├── __init__.py
│   ├── base.py                  # Generic repository
│   └── farmer_repository.py
└── tests/
    ├── __init__.py
    ├── conftest.py
    └── api/
        └── test_farmers.py
```

## Essential Libraries

```bash
# Core
uv add fastapi "uvicorn[standard]"

# Database (choose one)
uv add sqlalchemy asyncpg          # PostgreSQL async
uv add sqlmodel                    # SQLAlchemy + Pydantic hybrid

# Validation & Settings
uv add pydantic-settings           # Environment configuration

# Authentication
uv add "python-jose[cryptography]" # JWT tokens
uv add passlib[bcrypt]             # Password hashing

# HTTP Client (for external APIs)
uv add httpx                       # Async HTTP client

# Dev tools
uv add --dev pytest pytest-asyncio pytest-cov
uv add --dev ruff mypy             # Linting & type checking
uv add --dev pre-commit
```

## Code Patterns

### Settings (Pydantic)
```python
# config/settings.py
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    app_name: str = "My API"
    debug: bool = False
    database_url: str
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    class Config:
        env_file = ".env"

@lru_cache
def get_settings() -> Settings:
    return Settings()
```

### Database Setup (Async SQLAlchemy)
```python
# config/database.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker, declarative_base

from .settings import get_settings

settings = get_settings()
engine = create_async_engine(settings.database_url, echo=settings.debug)
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
Base = declarative_base()

async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
```

### Base Model
```python
# models/base.py
from sqlalchemy import Column, DateTime
from sqlalchemy.dialects.postgresql import UUID
from datetime import datetime
import uuid

from config.database import Base

class BaseModel(Base):
    __abstract__ = True

    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow, nullable=False)
```

### Pydantic Schema
```python
# schemas/farmer.py
from pydantic import BaseModel, ConfigDict
from uuid import UUID
from datetime import datetime

class FarmerBase(BaseModel):
    name: str
    phone: str
    location: str | None = None

class FarmerCreate(FarmerBase):
    pass

class FarmerUpdate(BaseModel):
    name: str | None = None
    phone: str | None = None
    location: str | None = None

class FarmerResponse(FarmerBase):
    id: UUID
    created_at: datetime
    updated_at: datetime

    model_config = ConfigDict(from_attributes=True)
```

### Service Layer
```python
# services/farmer_service.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from uuid import UUID

from models.farmer import Farmer
from schemas.farmer import FarmerCreate, FarmerUpdate

class FarmerService:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def get_all(self, skip: int = 0, limit: int = 100) -> list[Farmer]:
        result = await self.db.execute(
            select(Farmer).offset(skip).limit(limit)
        )
        return result.scalars().all()

    async def get_by_id(self, farmer_id: UUID) -> Farmer | None:
        result = await self.db.execute(
            select(Farmer).where(Farmer.id == farmer_id)
        )
        return result.scalar_one_or_none()

    async def create(self, data: FarmerCreate) -> Farmer:
        farmer = Farmer(**data.model_dump())
        self.db.add(farmer)
        await self.db.flush()
        await self.db.refresh(farmer)
        return farmer

    async def update(self, farmer_id: UUID, data: FarmerUpdate) -> Farmer | None:
        farmer = await self.get_by_id(farmer_id)
        if not farmer:
            return None
        for field, value in data.model_dump(exclude_unset=True).items():
            setattr(farmer, field, value)
        await self.db.flush()
        await self.db.refresh(farmer)
        return farmer
```

### API Endpoint
```python
# api/v1/endpoints/farmers.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from uuid import UUID

from config.database import get_db
from schemas.farmer import FarmerCreate, FarmerUpdate, FarmerResponse
from services.farmer_service import FarmerService

router = APIRouter(prefix="/farmers", tags=["farmers"])

def get_farmer_service(db: AsyncSession = Depends(get_db)) -> FarmerService:
    return FarmerService(db)

@router.get("", response_model=list[FarmerResponse])
async def list_farmers(
    skip: int = 0,
    limit: int = 100,
    service: FarmerService = Depends(get_farmer_service)
):
    return await service.get_all(skip=skip, limit=limit)

@router.get("/{farmer_id}", response_model=FarmerResponse)
async def get_farmer(
    farmer_id: UUID,
    service: FarmerService = Depends(get_farmer_service)
):
    farmer = await service.get_by_id(farmer_id)
    if not farmer:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Farmer not found")
    return farmer

@router.post("", response_model=FarmerResponse, status_code=status.HTTP_201_CREATED)
async def create_farmer(
    data: FarmerCreate,
    service: FarmerService = Depends(get_farmer_service)
):
    return await service.create(data)

@router.patch("/{farmer_id}", response_model=FarmerResponse)
async def update_farmer(
    farmer_id: UUID,
    data: FarmerUpdate,
    service: FarmerService = Depends(get_farmer_service)
):
    farmer = await service.update(farmer_id, data)
    if not farmer:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Farmer not found")
    return farmer
```

### Main Application
```python
# main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager

from config.settings import get_settings
from config.database import engine, Base
from api.v1.router import api_router

settings = get_settings()

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    # Shutdown
    await engine.dispose()

app = FastAPI(
    title=settings.app_name,
    lifespan=lifespan,
    docs_url="/docs",
    redoc_url="/redoc",
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configure for production
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(api_router, prefix="/api/v1")

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

### Exception Handling
```python
# core/exceptions.py
from fastapi import HTTPException, status

class NotFoundError(HTTPException):
    def __init__(self, detail: str = "Resource not found"):
        super().__init__(status_code=status.HTTP_404_NOT_FOUND, detail=detail)

class BadRequestError(HTTPException):
    def __init__(self, detail: str = "Bad request"):
        super().__init__(status_code=status.HTTP_400_BAD_REQUEST, detail=detail)

class UnauthorizedError(HTTPException):
    def __init__(self, detail: str = "Not authenticated"):
        super().__init__(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail=detail,
            headers={"WWW-Authenticate": "Bearer"},
        )
```

## Setup Commands

```bash
# Create project
uv init my-api && cd my-api

# Add dependencies
uv add fastapi "uvicorn[standard]"
uv add sqlalchemy asyncpg pydantic-settings
uv add "python-jose[cryptography]" "passlib[bcrypt]"
uv add --dev pytest pytest-asyncio ruff mypy

# Create structure
mkdir -p src/{config,api/v1/endpoints,core,models,schemas,services,repositories,tests/api}
touch src/{main.py,__init__.py}
touch src/config/{__init__.py,settings.py,database.py}
touch src/api/{__init__.py,deps.py}
touch src/api/v1/{__init__.py,router.py}

# Environment
echo "DATABASE_URL=postgresql+asyncpg://user:pass@localhost/dbname" > .env
echo "SECRET_KEY=$(openssl rand -hex 32)" >> .env

# Run
uv run uvicorn src.main:app --reload --host 0.0.0.0 --port 8000
```

## pyproject.toml

```toml
[project]
name = "my-api"
version = "0.1.0"
requires-python = ">=3.11"

[tool.ruff]
target-version = "py311"
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]

[tool.mypy]
python_version = "3.11"
strict = true
plugins = ["pydantic.mypy"]

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["src/tests"]
```

## Key Rules

- Use async/await everywhere - FastAPI is built for async
- Use Pydantic for ALL data validation and serialization
- Use dependency injection for services and database sessions
- Keep endpoints thin - business logic goes in services
- Use proper status codes (201 for created, 204 for no content, etc.)
- Document endpoints with docstrings (shows in OpenAPI)
