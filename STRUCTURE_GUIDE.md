# Quick Structure Guide

## Where to Find Things

### Documentation
| Need | Location |
|------|----------|
| Project overview | `docs/plans/01_PROJECT_OVERVIEW.md` |
| Frontend architecture | `docs/plans/02_FRONTEND_PLAN.md` |
| Backend architecture | `docs/plans/03_BACKEND_PLAN.md` |
| Test strategy | `docs/plans/04_TEST_STRATEGY.md` |
| E2E scenarios | `docs/plans/05_E2E_SCENARIOS.md` |
| Deepgram setup | `docs/setup/DEEPGRAM_VIETNAMESE_SETUP.md` |
| Sample audio | `docs/assets/` |
| Testing guide | `docs/misc/TESTING_GUIDE.md` |

### Backend Code
| Need | Location |
|------|----------|
| API routes | `backend/src/api/routes/` |
| Authentication | `backend/src/api/auth.py` |
| Database models | `backend/src/api/models.py` |
| STT service | `backend/src/services/stt/service.py` |
| Translation | `backend/src/services/translation/service.py` |
| NLP processing | `backend/src/services/nlp/` |
| Speaker diarization | `backend/src/services/diarization/service.py` |
| CrewAI agents | `backend/src/services/crewai/` |
| Cost tracking | `backend/src/services/cost/` |
| External providers | `backend/src/core/providers/` |
| Database migrations | `backend/database/migrations/` |
| Configuration | `backend/config/` |
| Sample data | `backend/data/` |
| Prompts | `backend/prompts/` |
| Database scripts | `backend/scripts/db/` |
| Dev scripts | `backend/scripts/dev/` |
| Tests | `backend/tests/` |

### Frontend Code
| Need | Location |
|------|----------|
| Pages | `frontend/src/pages/` |
| Components | `frontend/src/components/` |
| API services | `frontend/src/services/` |
| Custom hooks | `frontend/src/hooks/` |
| State management | `frontend/src/stores/` |
| Type definitions | `frontend/src/types/` |
| Utilities | `frontend/src/utils/` |

## Common Tasks

### Add a New Backend Service
1. Create folder in `backend/src/services/your-service/`
2. Implement `service.py`
3. Add route handler in `backend/src/api/routes/`
4. Update API schema in `backend/src/api/schemas.py`
5. Add tests in `backend/tests/your-service/`

### Add a New Frontend Page
1. Create page component in `frontend/src/pages/`
2. Create related components in `frontend/src/components/`
3. Add routing in frontend configuration
4. Add tests in `frontend/tests/pages/`

### Update Database Schema
1. Create migration script in `backend/database/migrations/`
2. Run migration script using database setup process
3. Update models in `backend/src/api/models.py`
4. Update schemas in `backend/src/api/schemas.py`

### Access External Data
1. Check `backend/src/core/providers/` for provider implementations
2. Use factory pattern in `backend/src/core/providers/factory.py`
3. Implement fallback/routing logic if needed

## File Organization Principles

1. **By Feature**: Services and features are grouped together
2. **By Layer**: API layer, services layer, core providers, database
3. **By Type**: Tests parallel the source structure
4. **Scalability**: Easy to add new services/features without disrupting existing code
5. **Clarity**: Clear folder names indicate purpose

## Updates After Reorganization

⚠️ **Important**: If you're using absolute imports or file references, verify they still work:

- Backend imports from `backend/src/stt` → now use `backend/src/services/stt`
- Backend imports from `backend/src/providers` → now use `backend/src/core/providers`
- Backend imports from `backend/src/{translation,nlp,diarization,crewai,cost}` → now use `backend/src/services/{service}`

Check `backend/SRC_STRUCTURE.md` for detailed module organization.
