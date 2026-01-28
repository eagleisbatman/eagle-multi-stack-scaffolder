# Flask Reference

## Research Queries
- "Flask best practices 2025 2026"
- "Flask project structure large applications"
- "Flask vs FastAPI vs Django 2025"
- "Flask SQLAlchemy setup 2025"
- "Flask application factory pattern"

## Package Manager
**uv** - 10-100x faster than pip, proper dependency resolution.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv init my-api && cd my-api
uv add flask
```

## When to Choose Flask

- Simple APIs or microservices
- Prototyping and MVPs
- When you want full control over architecture
- Lightweight applications
- Learning Python web development

**Note**: For high-performance async APIs, consider FastAPI instead.

## Project Structure

```
src/
├── app/
│   ├── __init__.py              # Application factory
│   ├── config.py                # Configuration classes
│   ├── extensions.py            # Flask extensions init
│   ├── api/
│   │   ├── __init__.py
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── users.py
│   │       ├── auth.py
│   │       └── farmers.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── user.py
│   │   └── farmer.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── farmer.py
│   ├── services/
│   │   ├── __init__.py
│   │   └── farmer_service.py
│   └── utils/
│       ├── __init__.py
│       └── errors.py
├── migrations/                   # Alembic migrations
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_farmers.py
├── .env
├── pyproject.toml
└── wsgi.py                      # WSGI entry point
```

## Essential Libraries

```bash
# Core
uv add flask

# Database
uv add flask-sqlalchemy           # SQLAlchemy integration
uv add flask-migrate              # Database migrations (Alembic)
uv add psycopg2-binary            # PostgreSQL driver

# API & Validation
uv add flask-restx                # REST API + Swagger docs
# OR
uv add marshmallow flask-marshmallow  # Serialization/validation

# Authentication
uv add flask-jwt-extended         # JWT authentication
uv add flask-bcrypt               # Password hashing

# CORS
uv add flask-cors                 # Cross-origin requests

# Dev tools
uv add --dev pytest pytest-flask
uv add --dev ruff mypy
uv add --dev python-dotenv        # .env loading
```

## Code Patterns

### Configuration
```python
# app/config.py
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    SECRET_KEY = os.getenv("SECRET_KEY", "dev-secret-key")
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class DevelopmentConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.getenv(
        "DATABASE_URL", "postgresql://localhost/myapp_dev"
    )

class ProductionConfig(Config):
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.getenv("DATABASE_URL")

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = "sqlite:///:memory:"

config = {
    "development": DevelopmentConfig,
    "production": ProductionConfig,
    "testing": TestingConfig,
    "default": DevelopmentConfig,
}
```

### Extensions
```python
# app/extensions.py
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_jwt_extended import JWTManager
from flask_bcrypt import Bcrypt
from flask_cors import CORS
from flask_marshmallow import Marshmallow

db = SQLAlchemy()
migrate = Migrate()
jwt = JWTManager()
bcrypt = Bcrypt()
cors = CORS()
ma = Marshmallow()
```

### Application Factory
```python
# app/__init__.py
from flask import Flask
from app.config import config
from app.extensions import db, migrate, jwt, bcrypt, cors, ma

def create_app(config_name: str = "default") -> Flask:
    app = Flask(__name__)
    app.config.from_object(config[config_name])

    # Initialize extensions
    db.init_app(app)
    migrate.init_app(app, db)
    jwt.init_app(app)
    bcrypt.init_app(app)
    cors.init_app(app)
    ma.init_app(app)

    # Register blueprints
    from app.api.v1 import api_v1_bp
    app.register_blueprint(api_v1_bp, url_prefix="/api/v1")

    # Health check
    @app.route("/health")
    def health():
        return {"status": "healthy"}

    # Error handlers
    from app.utils.errors import register_error_handlers
    register_error_handlers(app)

    return app
```

### Base Model
```python
# app/models/base.py
from datetime import datetime
import uuid
from app.extensions import db

class BaseModel(db.Model):
    __abstract__ = True

    id = db.Column(db.String(36), primary_key=True, default=lambda: str(uuid.uuid4()))
    created_at = db.Column(db.DateTime, default=datetime.utcnow, nullable=False)
    updated_at = db.Column(
        db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow, nullable=False
    )

    def save(self):
        db.session.add(self)
        db.session.commit()
        return self

    def delete(self):
        db.session.delete(self)
        db.session.commit()
```

### Model
```python
# app/models/farmer.py
from app.models.base import BaseModel
from app.extensions import db

class Farmer(BaseModel):
    __tablename__ = "farmers"

    name = db.Column(db.String(100), nullable=False)
    phone = db.Column(db.String(20), nullable=False)
    location = db.Column(db.String(200))

    def __repr__(self):
        return f"<Farmer {self.name}>"
```

### Schema (Marshmallow)
```python
# app/schemas/farmer.py
from app.extensions import ma
from app.models.farmer import Farmer
from marshmallow import fields, validate

class FarmerSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Farmer
        load_instance = True
        include_fk = True

    id = fields.String(dump_only=True)
    created_at = fields.DateTime(dump_only=True)
    updated_at = fields.DateTime(dump_only=True)
    name = fields.String(required=True, validate=validate.Length(min=1, max=100))
    phone = fields.String(required=True, validate=validate.Length(min=1, max=20))
    location = fields.String(validate=validate.Length(max=200))

class FarmerUpdateSchema(ma.Schema):
    name = fields.String(validate=validate.Length(min=1, max=100))
    phone = fields.String(validate=validate.Length(min=1, max=20))
    location = fields.String(validate=validate.Length(max=200))

farmer_schema = FarmerSchema()
farmers_schema = FarmerSchema(many=True)
farmer_update_schema = FarmerUpdateSchema()
```

### Service Layer
```python
# app/services/farmer_service.py
from app.models.farmer import Farmer
from app.extensions import db

class FarmerService:
    @staticmethod
    def get_all(page: int = 1, per_page: int = 20):
        return Farmer.query.paginate(page=page, per_page=per_page, error_out=False)

    @staticmethod
    def get_by_id(farmer_id: str) -> Farmer | None:
        return Farmer.query.get(farmer_id)

    @staticmethod
    def create(data: dict) -> Farmer:
        farmer = Farmer(**data)
        return farmer.save()

    @staticmethod
    def update(farmer: Farmer, data: dict) -> Farmer:
        for key, value in data.items():
            if value is not None:
                setattr(farmer, key, value)
        db.session.commit()
        return farmer

    @staticmethod
    def delete(farmer: Farmer) -> None:
        farmer.delete()
```

### API Blueprint
```python
# app/api/v1/__init__.py
from flask import Blueprint

api_v1_bp = Blueprint("api_v1", __name__)

from app.api.v1 import farmers, users, auth  # noqa
```

### API Endpoint
```python
# app/api/v1/farmers.py
from flask import request, jsonify
from app.api.v1 import api_v1_bp
from app.services.farmer_service import FarmerService
from app.schemas.farmer import farmer_schema, farmers_schema, farmer_update_schema
from app.utils.errors import NotFoundError, BadRequestError

@api_v1_bp.route("/farmers", methods=["GET"])
def list_farmers():
    page = request.args.get("page", 1, type=int)
    per_page = request.args.get("per_page", 20, type=int)
    pagination = FarmerService.get_all(page=page, per_page=per_page)
    return jsonify({
        "data": farmers_schema.dump(pagination.items),
        "meta": {
            "page": pagination.page,
            "per_page": pagination.per_page,
            "total": pagination.total,
            "pages": pagination.pages,
        }
    })

@api_v1_bp.route("/farmers/<farmer_id>", methods=["GET"])
def get_farmer(farmer_id: str):
    farmer = FarmerService.get_by_id(farmer_id)
    if not farmer:
        raise NotFoundError("Farmer not found")
    return jsonify(farmer_schema.dump(farmer))

@api_v1_bp.route("/farmers", methods=["POST"])
def create_farmer():
    data = request.get_json()
    errors = farmer_schema.validate(data)
    if errors:
        raise BadRequestError(errors)
    farmer = FarmerService.create(data)
    return jsonify(farmer_schema.dump(farmer)), 201

@api_v1_bp.route("/farmers/<farmer_id>", methods=["PATCH"])
def update_farmer(farmer_id: str):
    farmer = FarmerService.get_by_id(farmer_id)
    if not farmer:
        raise NotFoundError("Farmer not found")
    data = request.get_json()
    errors = farmer_update_schema.validate(data)
    if errors:
        raise BadRequestError(errors)
    updated = FarmerService.update(farmer, data)
    return jsonify(farmer_schema.dump(updated))

@api_v1_bp.route("/farmers/<farmer_id>", methods=["DELETE"])
def delete_farmer(farmer_id: str):
    farmer = FarmerService.get_by_id(farmer_id)
    if not farmer:
        raise NotFoundError("Farmer not found")
    FarmerService.delete(farmer)
    return "", 204
```

### Error Handling
```python
# app/utils/errors.py
from flask import jsonify, Flask

class APIError(Exception):
    def __init__(self, message, status_code=400):
        self.message = message
        self.status_code = status_code

class NotFoundError(APIError):
    def __init__(self, message="Resource not found"):
        super().__init__(message, 404)

class BadRequestError(APIError):
    def __init__(self, message="Bad request"):
        super().__init__(message, 400)

class UnauthorizedError(APIError):
    def __init__(self, message="Unauthorized"):
        super().__init__(message, 401)

def register_error_handlers(app: Flask):
    @app.errorhandler(APIError)
    def handle_api_error(error):
        return jsonify({"error": error.message}), error.status_code

    @app.errorhandler(404)
    def handle_404(error):
        return jsonify({"error": "Not found"}), 404

    @app.errorhandler(500)
    def handle_500(error):
        return jsonify({"error": "Internal server error"}), 500
```

### WSGI Entry Point
```python
# wsgi.py
import os
from app import create_app

config_name = os.getenv("FLASK_ENV", "development")
app = create_app(config_name)

if __name__ == "__main__":
    app.run()
```

## Setup Commands

```bash
# Create project
uv init my-api && cd my-api

# Add dependencies
uv add flask flask-sqlalchemy flask-migrate psycopg2-binary
uv add flask-jwt-extended flask-bcrypt flask-cors
uv add marshmallow flask-marshmallow
uv add --dev pytest pytest-flask ruff mypy python-dotenv

# Create structure
mkdir -p app/{api/v1,models,schemas,services,utils}
mkdir -p tests migrations
touch app/{__init__.py,config.py,extensions.py}
touch app/api/{__init__.py}
touch app/api/v1/{__init__.py,farmers.py}
touch wsgi.py

# Environment
cat > .env << EOF
FLASK_ENV=development
SECRET_KEY=$(openssl rand -hex 32)
DATABASE_URL=postgresql://user:pass@localhost/dbname
EOF

# Initialize database
uv run flask --app wsgi:app db init
uv run flask --app wsgi:app db migrate -m "Initial migration"
uv run flask --app wsgi:app db upgrade

# Run development server
uv run flask --app wsgi:app run --debug --host 0.0.0.0 --port 5000

# Production (with gunicorn)
uv add gunicorn
uv run gunicorn -w 4 -b 0.0.0.0:5000 wsgi:app
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
warn_return_any = true
warn_unused_configs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
```

## Key Rules

- ALWAYS use the application factory pattern
- ALWAYS use blueprints for organizing routes
- Use Flask-Migrate for database migrations (never raw SQL DDL)
- Keep views thin - business logic goes in services
- Use Marshmallow for validation and serialization
- Use environment variables for all configuration
- For production, use Gunicorn (not Flask's dev server)
