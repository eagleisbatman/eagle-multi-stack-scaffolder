# Django Reference

## Research Queries
- "Django best practices 2025 2026"
- "Django REST Framework vs Django Ninja 2025"
- "Django project structure large applications"
- "uv Python package manager Django"

## Package Manager
**uv** - 10-100x faster than pip, proper dependency resolution.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv init my-project && cd my-project
uv add django djangorestframework
```

## Project Structure

```
src/
├── {project}/
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── apps/
│   ├── core/
│   │   ├── models.py       # Base models
│   │   ├── permissions.py
│   │   └── pagination.py
│   ├── users/
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   └── services.py
│   └── farmers/
│       ├── models.py
│       ├── serializers.py
│       ├── views.py
│       ├── services.py
│       └── selectors.py
└── manage.py
```

## Essential Libraries

```bash
uv add django djangorestframework django-filter drf-spectacular
uv add "psycopg[binary]"              # PostgreSQL
uv add django-cors-headers            # CORS
uv add python-decouple                # Config
uv add celery redis                   # Tasks
uv add --dev pytest pytest-django ruff mypy
```

## Code Patterns

### Base Model
```python
from django.db import models
import uuid

class TimeStampedModel(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True
```

### Service Layer
```python
# services.py
from django.db import transaction

class FarmerService:
    @staticmethod
    @transaction.atomic
    def create(*, name: str, phone: str) -> Farmer:
        return Farmer.objects.create(name=name, phone=phone)
```

### ViewSet
```python
from rest_framework import viewsets
from .models import Farmer
from .serializers import FarmerSerializer

class FarmerViewSet(viewsets.ModelViewSet):
    queryset = Farmer.objects.all()
    serializer_class = FarmerSerializer
    filterset_fields = ['location']
    search_fields = ['name', 'phone']
```

## DRF Settings

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}
```

## Setup Commands

```bash
uv init my-project && cd my-project
uv add django djangorestframework

uv run django-admin startproject config src
cd src
uv run python manage.py startapp core
uv run python manage.py startapp farmers

uv run python manage.py migrate
uv run python manage.py createsuperuser
uv run python manage.py runserver
```
