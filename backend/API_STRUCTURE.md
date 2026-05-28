# Backend API Structure Guide

## Overview
The API is organized into logical modules by responsibility, making it easier to navigate and maintain.

## Directory Structure

```
backend/src/api/
├── middleware/              # Request/response processing
│   ├── logging_middleware.py
│   ├── maintenance_middleware.py
│   └── rate_limiting.py
│
├── models/                  # Data models and schemas
│   ├── models.py           # SQLAlchemy models
│   └── schemas.py          # Pydantic schemas
│
├── database/               # Database configuration
│   ├── database.py         # DB setup & connection
│   └── persistence.py      # Data persistence utilities
│
├── utils/                  # Utility functions
│   ├── sanitization.py     # Input sanitization
│   ├── storage.py          # File storage operations
│   └── export.py           # Data export functionality
│
├── services/               # Business logic services
│   └── notifications.py    # Notification handling
│
├── jobs/                   # Background job handling
│   ├── jobs.py            # Job definitions
│   └── scheduler.py        # Job scheduling
│
├── crud/                   # Database operations
│   ├── crud_user.py
│   ├── crud_meeting.py
│   ├── crud_organization.py
│   ├── crud_group.py
│   ├── crud_ai_data.py
│   └── crud_system.py
│
├── routes/                 # API endpoints
│   ├── auth_profile.py
│   ├── meetings.py
│   ├── action_items.py
│   ├── organizations.py
│   ├── groups.py
│   ├── analytics.py
│   ├── notifications.py
│   ├── search.py
│   ├── stt.py
│   ├── jobs.py
│   ├── admin.py
│   └── system.py
│
├── core/                   # Core business logic
│   ├── meeting_operations.py
│   ├── user_payloads.py
│   ├── action_item_operations.py
│   ├── analytics_operations.py
│   └── [20+ operation files]
│
├── main.py                # FastAPI application setup
├── config.py              # Configuration
├── app.py                 # App initialization
├── auth.py                # Authentication logic
├── chat.py                # Chat functionality
└── swagger.py             # Swagger documentation

```

## Module Purposes

| Module | Purpose |
|--------|---------|
| **middleware/** | Request processing, logging, rate limiting |
| **models/** | Data schemas and ORM models |
| **database/** | Database connection and persistence |
| **utils/** | Helper functions (sanitization, storage, export) |
| **services/** | Business logic (notifications, etc) |
| **jobs/** | Background jobs and scheduling |
| **crud/** | Database CRUD operations |
| **routes/** | API endpoint definitions |
| **core/** | Complex operation handlers |

## Import Examples

```python
# From middleware
from backend.src.api.middleware.logging_middleware import LoggingMiddleware

# From models
from backend.src.api.models.models import User, Meeting

# From utils
from backend.src.api.utils.sanitization import sanitize_input
from backend.src.api.utils.storage import save_file

# From services
from backend.src.api.services.notifications import send_notification

# From crud
from backend.src.api.crud.crud_user import get_user

# From routes
from backend.src.api.routes.meetings import router as meetings_router

# From core
from backend.src.api.core.meeting_operations import process_meeting
```

## Adding New Features

1. **New API Endpoint**: Add to `routes/` (e.g., `routes/new_feature.py`)
2. **New Operation**: Add to `core/` (e.g., `core/new_feature_operations.py`)
3. **New Service**: Add to `services/` (e.g., `services/new_service.py`)
4. **DB Access**: Add to `crud/crud_*.py` or create new CRUD file
5. **Utility Function**: Add to `utils/`

## Import Path Updates

If you see import errors after reorganization, update paths:

```python
# OLD
from api.logging_middleware import LoggingMiddleware

# NEW
from api.middleware.logging_middleware import LoggingMiddleware

# OLD
from api.models import User

# NEW
from api.models.models import User
```
