# Backend Source Structure

## `/src/api/` - Main API Application
- `app.py` - FastAPI application initialization
- `main.py` - Entry point
- `auth.py` - Authentication logic
- `chat.py` - Chat endpoints
- `config.py` - API configuration
- `database.py` - Database connection setup
- `models.py` - Data models
- `schemas.py` - Pydantic schemas
- `notifications.py` - Notification system
- `jobs.py` - Background jobs
- `rate_limiting.py` - Rate limiting middleware
- `logging_middleware.py` - Logging middleware
- `maintenance_middleware.py` - Maintenance middleware
- `export.py` - Export functionality
- `sanitization.py` - Input sanitization
- `scheduler.py` - Task scheduling
- `storage.py` - File storage
- `swagger.py` - Swagger documentation
- `persistence.py` - Data persistence layer

### `/api/core/` - Core Business Logic
- `/core/providers/` - External service providers
  - `deepgram.py` - Deepgram STT provider
  - `google_llm.py` - Google LLM provider
  - `llm.py` - Base LLM interface
  - `router_llm.py` - LLM routing
  - `nlp_eval.py` - NLP evaluation
  - `errors.py` - Provider error handling
  - `factory.py` - Provider factory

### `/api/routes/` - API Route Handlers
- Organized by feature/endpoint

### `/api/crud/` - Database Operations
- CRUD operations for database entities

## `/src/services/` - Business Logic Services

### `/services/stt/` - Speech-to-Text
- `service.py` - STT service implementation

### `/services/translation/` - Translation Service
- `service.py` - Translation logic

### `/services/nlp/` - Natural Language Processing
- `phobert_processor.py` - PhoBERT NLP processing
- `bartpho_enhancer.py` - BARTpho enhancement
- `context_corrector.py` - Context correction
- `dialect_classifier.py` - Dialect classification

### `/services/diarization/` - Speaker Diarization
- `service.py` - Diarization service

### `/services/crewai/` - CrewAI Agents
- `agents.py` - Agent definitions
- `orchestrator.py` - Agent orchestration
- `router.py` - Agent routing
- `tools.py` - Agent tools

### `/services/cost/` - Cost Management
- `cost_logger.py` - Cost logging
- `alert_service.py` - Cost alerts
- `api.py` - Cost API

## `/config/` - Configuration Files
- `/agents/` - Agent configuration files

## `/database/` - Database Setup
- `/migrations/` - Database migration scripts
  - `mysql_schema.sql` - MySQL schema definition

## `/data/` - Sample & Test Data
- `sample_meeting_vi.wav` - Sample audio file
- `meetings.json` - Sample meeting data
- `admin_settings.json` - Admin settings
- `admin_prompts.json` - Admin prompts
- `ab_test_dataset.jsonl` - A/B test data

## `/prompts/` - AI Prompts
- `translation_prompt_v1.yaml` - Translation prompt v1
- `translation_prompt_v2.yaml` - Translation prompt v2
- `translation_prompt_v3.yaml` - Translation prompt v3

## `/scripts/` - Utility Scripts

### `/scripts/db/` - Database Scripts
- `init_mysql.py` - Initialize MySQL database
- `migrate_json_to_mysql.py` - Migrate JSON data to MySQL
- `add_phobert_columns.py` - Add PhoBERT columns
- `add_reply_to_column.py` - Add reply column
- `migrate_auth_v2.py` - Migrate auth v2
- `migrate_notifications.py` - Migrate notifications
- `seed_db.py` - Seed database with test data

### `/scripts/dev/` - Development Scripts
- `benchmark_stt.py` - STT benchmarking
- `run_prompt_ab.py` - A/B test prompts
- `train_ai_prompts.py` - Train AI prompts

### `/scripts/ops/` - Operations Scripts
- `run_r3_*.py` - Various smoke tests

## `/tests/` - Test Suite
- Tests organized by module mirroring `/src/` structure
- `/tests/api/` - API tests
- `/tests/contracts/` - Contract tests
- `/tests/crud/` - CRUD tests
- `/tests/routes/` - Route tests
- `/tests/services/` - Service tests
- `/tests/providers/` - Provider tests
- `/tests/smoke/` - Smoke tests
- `/tests/manual/` - Manual test runners
