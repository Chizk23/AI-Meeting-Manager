# AI Meeting Manager - Project Structure (Reorganized)

## Overview
This document provides a guide to the reorganized project structure. The project is now organized with clear separation between frontend, backend, documentation, and configuration.

## Root Directory Structure

```
AI-Meeting-Manager/
├── docs/                          # Project documentation (reorganized)
│   ├── README.md                  # Documentation index
│   ├── plans/                     # Project planning documents
│   │   ├── 01_PROJECT_OVERVIEW.md
│   │   ├── 02_FRONTEND_PLAN.md
│   │   ├── 03_BACKEND_PLAN.md
│   │   ├── 04_TEST_STRATEGY.md
│   │   ├── 05_E2E_SCENARIOS.md
│   │   ├── WORKLOG_2026-05-25.md
│   │   └── _PLANS_README.md
│   ├── setup/                     # Setup & configuration guides
│   │   ├── DEEPGRAM_QUICK_REFERENCE.md
│   │   └── DEEPGRAM_VIETNAMESE_SETUP.md
│   ├── assets/                    # Media files (audio, images)
│   │   ├── Nguyen-Cao-3.wav
│   │   └── suthabantre.mp3
│   └── misc/                      # Miscellaneous documentation
│       ├── GEMINI.md
│       └── TESTING_GUIDE.md
│
├── backend/                       # Python/FastAPI backend
│   ├── src/
│   │   ├── api/                   # Main API application
│   │   │   ├── core/              # Core business logic & providers
│   │   │   ├── routes/            # API route handlers
│   │   │   ├── crud/              # Database CRUD operations
│   │   │   └── [config, auth, models, schemas, etc.]
│   │   └── services/              # Business logic services
│   │       ├── stt/               # Speech-to-Text service
│   │       ├── translation/       # Translation service
│   │       ├── nlp/               # NLP processing
│   │       ├── diarization/       # Speaker diarization
│   │       ├── crewai/            # CrewAI agents
│   │       └── cost/              # Cost management
│   ├── config/
│   │   └── agents/                # Agent configuration files
│   ├── database/
│   │   └── migrations/            # Database migration scripts
│   ├── data/                      # Sample & test data
│   ├── prompts/                   # AI prompt templates
│   ├── scripts/                   # Utility scripts
│   │   ├── db/                    # Database management scripts
│   │   ├── dev/                   # Development scripts
│   │   └── ops/                   # Operations/testing scripts
│   ├── tests/                     # Test suite
│   ├── SRC_STRUCTURE.md           # Detailed backend structure guide
│   ├── requirements.txt
│   └── Dockerfile
│
├── frontend/                      # React/Vite frontend
│   ├── src/
│   │   ├── components/            # Reusable UI components
│   │   ├── pages/                 # Page components
│   │   ├── services/              # API services
│   │   ├── hooks/                 # Custom React hooks
│   │   ├── context/               # React context providers
│   │   ├── stores/                # State management
│   │   ├── utils/                 # Utility functions
│   │   ├── types/                 # TypeScript type definitions
│   │   └── [assets, lib, etc.]
│   ├── public/                    # Static assets
│   ├── tests/                     # Frontend tests
│   └── package.json
│
├── archive/                       # Archived/legacy code
│   └── backend-scratch/           # Old scratch files
│
├── external_tools/                # External integrations
│   └── everything-claude-code/    # Claude code tool
│
├── docker-compose.yml             # Docker Compose configuration
├── nginx.conf                     # Nginx configuration
├── README.md                      # Main project README
├── .gitignore
└── vitest.config.ts              # Vitest configuration

```

## Key Changes Made

### ✅ Documentation Organization
- Moved all documentation to `/docs/` folder
- Created `/docs/plans/` for project planning documents
- Created `/docs/setup/` for integration guides (Deepgram, etc.)
- Created `/docs/assets/` for media files (audio, images)
- Created `/docs/misc/` for miscellaneous docs
- Added `docs/README.md` as documentation index

### ✅ Backend Service Organization
- Consolidated scattered services into `/backend/src/services/`
  - `stt/` - Speech-to-text services
  - `translation/` - Translation services
  - `nlp/` - NLP processing services
  - `diarization/` - Speaker diarization
  - `crewai/` - CrewAI agent orchestration
  - `cost/` - Cost tracking and management
- Organized providers into `/backend/src/core/providers/`
- Database migrations moved to `/backend/database/migrations/`
- Agent configs in `/backend/config/agents/`

### ✅ Root Level Cleanup
- Removed `AUDIO/` folder → files moved to `/docs/assets/`
- Removed `PLANS/` folder → files moved to `/docs/plans/`
- Removed scattered root-level doc files → organized into `/docs/`
- Added `PROJECT_STRUCTURE.md` as structure guide

### ✅ New Guide Files
- `docs/README.md` - Documentation index
- `backend/SRC_STRUCTURE.md` - Detailed backend structure guide

## Navigation Tips

1. **Project Overview**: Read `docs/plans/01_PROJECT_OVERVIEW.md`
2. **Backend Development**: See `backend/SRC_STRUCTURE.md` for detailed organization
3. **Frontend Setup**: Check `frontend/README.md`
4. **Setup Guides**: Look in `docs/setup/` for integration guides
5. **Testing**: See `docs/misc/TESTING_GUIDE.md`

## Important Notes

- `.gitkeep` files are placed in empty directories to maintain folder structure
- All imports in backend code may need verification after this reorganization
- The `external_tools/` directory contains external integrations and should be carefully handled
- The `archive/` directory contains legacy code and can be referenced but shouldn't be used in active development
