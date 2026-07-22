# KIẾN TRÚC HỆ THỐNG MULTIMINUTES AI (CONVIA)

## Mục Lục
1. [Tổng Quan Kiến Trúc](#tổng-quan-kiến-trúc)
2. [Backend Architecture](#backend-architecture)
3. [Frontend Architecture](#frontend-architecture)
4. [Database Schema](#database-schema)
5. [API Design](#api-design)
6. [Data Flow](#data-flow)
7. [Deployment Architecture](#deployment-architecture)

---

## Tổng Quan Kiến Trúc

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER (React 19)                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │Dashboard │ │Meetings  │ │ Calendar │ │  Admin   │           │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘           │
│       └─────────────┼──────────────┼──────────┘                 │
└────────────────────┼──────────────┼──────────────────────────────┘
                     │ Axios + TanStack Query
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│         API GATEWAY (FastAPI 1.0 + 13 Routers)                   │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │Organizations│ │  Meetings    │ │    Admin     │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │  Auth       │ │    STT       │ │  Analytics   │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
└────────────────────┬──────────────────────────────────────────────┘
                     │
    ┌────────────────┼────────────────┬────────────────┐
    ▼                ▼                ▼                ▼
┌─────────────┐ ┌──────────┐  ┌──────────────┐  ┌──────────┐
│   MySQL 8   │ │CrewAI    │  │ AI Providers │  │  Redis   │
│  (24 Entities)│ (15 Agents)│  │(Deepgram,   │  │ (Cache)  │
│             │ │         │  │ Groq, etc)   │  │          │
└─────────────┘ └──────────┘  └──────────────┘  └──────────┘
```

### Layers

```
┌─────────────────────────────────────────────────────────────┐
│  UI LAYER                 (React Components)                 │
│  - Pages, Features, Components                              │
├─────────────────────────────────────────────────────────────┤
│  STATE LAYER              (Zustand + React Query)            │
│  - Client State, Server State, Auth Context                 │
├─────────────────────────────────────────────────────────────┤
│  API LAYER                (Axios + Services)                 │
│  - API Calls, Mappers, Type Conversion                      │
├─────────────────────────────────────────────────────────────┤
│  BACKEND LAYER            (FastAPI)                          │
│  - Routes, Schemas, Auth Middleware                          │
├─────────────────────────────────────────────────────────────┤
│  BUSINESS LOGIC LAYER     (Core Operations)                  │
│  - CRUD, Permissions, AI Pipeline                            │
├─────────────────────────────────────────────────────────────┤
│  DATA LAYER               (SQLAlchemy + MySQL)              │
│  - 24 Database Entities, Relationships, Queries              │
├─────────────────────────────────────────────────────────────┤
│  EXTERNAL LAYER           (AI Providers + Services)          │
│  - Deepgram, Groq, PhoBERT, BARTpho, CrewAI                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Backend Architecture

### 1. FastAPI Setup (app.py)

```python
# File: backend/src/api/app.py

def create_app() -> FastAPI:
    app = FastAPI(
        title="CONVIA API",
        version="1.0.0-beta",
        lifespan=lifespan,  # Lifecycle management
    )
    
    # Middleware Stack (order matters!)
    app.add_middleware(RequestLoggingMiddleware)      # 1. Log requests
    app.add_middleware(MaintenanceModeMiddleware)     # 2. Maintenance check
    app.add_middleware(RateLimitMiddleware)           # 3. Rate limiting (100 RPM)
    app.add_middleware(CORSMiddleware)                # 4. CORS policy
    
    # 13 Routers
    app.include_router(auth_profile_router)           # Login, Profile
    app.include_router(organizations_router)         # Org CRUD
    app.include_router(groups_router)                # Group CRUD
    app.include_router(meetings_router)              # Meeting CRUD
    app.include_router(stt_router)                   # STT + Transcripts
    app.include_router(admin_router)                 # Admin Panel
    app.include_router(action_items_router)          # Action Items
    app.include_router(system_router)                # Health + Diagnostics
    app.include_router(notifications_router)        # Notifications
    app.include_router(analytics_router)             # Analytics + Reports
    app.include_router(search_router)                # Full-text Search
    app.include_router(jobs_router)                  # Job Queue Monitoring
    app.include_router(export_router)                # PDF/DOCX/CSV Export
    
    return app
```

### 2. Request/Response Flow

```
Request
   ↓
CORSMiddleware (check origin)
   ↓
RateLimitMiddleware (100 RPM check)
   ↓
MaintenanceModeMiddleware (maintenance mode?)
   ↓
RequestLoggingMiddleware (log to file/DB)
   ↓
Route Handler
   ├─ Extract params/body
   ├─ Validate with Pydantic schema
   ├─ get_current_user dependency (JWT token)
   ├─ Check permissions (RBAC)
   ├─ Call business logic (core/*.py)
   └─ Return response (Pydantic model)
      ↓
Response Headers (CORS, Cache-Control)
   ↓
JSON Response (serialized via model.model_dump())
```

### 3. 13 Routers (API Routes)

| Router | Endpoint | Purpose | Key Endpoints |
|--------|----------|---------|---------------|
| **auth_profile** | `/api/auth/`, `/api/profile/` | Login, Register, Password | POST /auth/login, POST /auth/register |
| **organizations** | `/api/organizations/` | Org CRUD | GET/POST/PUT/DELETE /orgs |
| **groups** | `/api/groups/` | Group CRUD | GET/POST /groups, PUT /groups/{id} |
| **meetings** | `/api/meetings/` | Meeting CRUD | GET/POST /meetings, GET /meetings/{id} |
| **stt** | `/api/stt/`, `/ws/meeting/` | STT + WebSocket | POST /stt/upload, WS /ws/meeting/{id}/stt |
| **admin** | `/api/admin/` | System Admin | GET /admin/users, POST /admin/orgs/approve |
| **action_items** | `/api/action-items/` | Task Management | GET/POST /action-items |
| **system** | `/api/health/`, `/api/system/` | Health + Config | GET /health, GET /system/config |
| **notifications** | `/api/notifications/` | Alerts | GET/POST /notifications |
| **analytics** | `/api/analytics/` | Dashboards | GET /analytics/costs, GET /analytics/meetings |
| **search** | `/api/search/` | Full-text Search | POST /search (meetings + transcripts) |
| **jobs** | `/api/jobs/` | Job Queue | GET /jobs/{id}/status |
| **export** | `/api/meetings/{id}/export` | Export | GET /export?format=pdf/docx/csv |

### 4. Business Logic Layer (core/*.py)

```
backend/src/api/core/
├── organization_operations.py       # Org create/update/list/delete
├── group_operations.py              # Group CRUD + members
├── meeting_operations.py            # Meeting CRUD + WebSocket manager
├── action_item_operations.py        # Action items CRUD
├── admin_operations.py              # System admin operations
├── analytics_operations.py          # Analytics + reports
├── auth_profile_operations.py       # Auth + password reset
├── notification_operations.py       # Notifications
├── search_operations.py             # Full-text search
├── system_operations.py             # Health + config
│
├── upload_jobs.py                   # Job queue (in-memory, Phase 2)
├── meeting_stt.py                   # STT settings
├── stt_support.py                   # STT logic + Deepgram
├── transcript_support.py            # Transcript building
├── nlp_support.py                   # PhoBERT + BARTpho
├── job_operations.py                # Job monitoring
│
├── app_state.py                     # Global config + logger
├── lifecycle.py                     # Startup/shutdown hooks
└── ...
```

**Pattern**: `*_operations.py` exports payload functions
```python
# Example: backend/src/api/core/organization_operations.py
def create_organization_payload(
    org_data: schemas.OrganizationCreate,
    db: Session,
    current_user: User
) -> Dict[str, Any]:
    """
    Create organization + set current_user as org-admin
    Returns: organization dict
    """
    # 1. Validate permission (current_user must be system-admin)
    if current_user.role != "system-admin":
        raise HTTPException(403, "Only system admin can create orgs")
    
    # 2. Create org
    org = Organization(
        name=org_data.name,
        description=org_data.description
    )
    db.add(org)
    db.flush()  # Get org.id
    
    # 3. Add current_user as org-admin
    user_org = UserOrganization(
        user_id=current_user.id,
        organization_id=org.id,
        role="org-admin"
    )
    db.add(user_org)
    db.commit()
    
    return schema.Organization.model_validate(org).model_dump()
```

### 5. CRUD Layer (crud/*.py)

```
backend/src/api/crud/
├── crud_user.py          # User create/read/update/delete
├── crud_organization.py  # Organization operations
├── crud_group.py         # Group operations
├── crud_meeting.py       # Meeting operations
├── crud_ai_data.py       # AI data (summary, processed)
└── crud_system.py        # System configuration
```

**Pattern**: Standard CRUD operations
```python
# Example: crud_organization.py
def create_organization(db: Session, org_data: schemas.OrganizationCreate) -> Organization:
    org = Organization(**org_data.model_dump())
    db.add(org)
    db.commit()
    db.refresh(org)
    return org

def get_organization(db: Session, org_id: str) -> Optional[Organization]:
    return db.query(Organization).filter(Organization.id == org_id).first()

def list_organizations(db: Session, skip: int = 0, limit: int = 100) -> List[Organization]:
    return db.query(Organization).offset(skip).limit(limit).all()

def update_organization(db: Session, org_id: str, org_data: schemas.OrganizationUpdate) -> Organization:
    org = get_organization(db, org_id)
    if org:
        for key, value in org_data.model_dump(exclude_unset=True).items():
            setattr(org, key, value)
        db.commit()
        db.refresh(org)
    return org

def delete_organization(db: Session, org_id: str) -> bool:
    org = get_organization(db, org_id)
    if org:
        db.delete(org)
        db.commit()
        return True
    return False
```

### 6. Authentication & Authorization

```python
# File: backend/src/api/auth.py

def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> User:
    """
    JWT token validation dependency
    Used: @app.get() depends on this
    """
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        user_id = payload.get("sub")
        if not user_id:
            raise HTTPException(401, "Invalid token")
    except jwt.InvalidTokenError:
        raise HTTPException(401, "Invalid token")
    
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(404, "User not found")
    
    return user

def check_org_access(
    user: User,
    org_id: str,
    db: Session,
    required_role: Optional[str] = None
) -> bool:
    """
    Check if user has access to organization
    required_role: "org-admin" or "member" or None (any role)
    """
    user_org = db.query(UserOrganization).filter(
        UserOrganization.user_id == user.id,
        UserOrganization.organization_id == org_id
    ).first()
    
    if not user_org:
        return False  # No access
    
    if required_role and user_org.role != required_role:
        return False  # Has access but wrong role
    
    return True

def check_group_access(
    user: User,
    group_id: str,
    db: Session,
    required_role: Optional[str] = None
) -> bool:
    """
    Check if user has access to group
    required_role: "group-admin", "member", "viewer"
    """
    membership = db.query(GroupMembership).filter(
        GroupMembership.user_id == user.id,
        GroupMembership.group_id == group_id
    ).first()
    
    if not membership:
        return False  # No access
    
    if required_role and membership.role != required_role:
        return False  # Has access but wrong role
    
    return True
```

### 7. AI Providers Integration

```
backend/src/providers/
├── deepgram.py           # STT (Speech-to-Text)
│   └─ Provider: Deepgram Nova-3
│   └─ Streaming via WebSocket
│
├── router_llm.py         # LLM Router
│   ├─ Primary: Groq (OpenAI-compatible)
│   ├─ Fallback: Google Gemini
│   └─ Fallback: OpenAI
│
├── google_llm.py         # Google Gemini
├── llm.py                # Base LLM interface
├── viwhisper.py          # ViWhisper (local STT fallback)
│
├── diarization.py        # Speaker diarization
├── nlp_eval.py           # NLP evaluation
├── errors.py             # Provider errors
└── factory.py            # Provider factory pattern

backend/src/nlp/
├── phobert_processor.py   # PhoBERT (embedding + tone)
├── bartpho_enhancer.py    # BARTpho (text generation)
├── context_corrector.py   # Groq context correction
├── dialect_classifier.py  # Vietnamese dialect detection
└── ...

backend/src/crewai/
├── agents.py             # 15 AI agents definition
├── tools.py              # Tools for agents
├── orchestrator.py       # Agent orchestration
└── router.py             # CrewAI routing
```

**Provider Factory Pattern:**
```python
# File: backend/src/providers/factory.py
class LLMFactory:
    @staticmethod
    def create_llm(provider: str = None):
        provider = provider or os.getenv("LLM_PROVIDER", "groq")
        
        if provider == "groq":
            return RouterLLMAdapter("groq")  # Groq (primary)
        elif provider == "google":
            return GoogleLLM()                # Gemini (fallback)
        elif provider == "openai":
            return OpenAILLM()                # OpenAI (fallback)
        else:
            raise ValueError(f"Unknown provider: {provider}")

# Usage:
llm = LLMFactory.create_llm()
summary = await llm.generate_summary(transcript)
```

### 8. WebSocket Real-time STT

```python
# File: backend/src/api/routes/stt.py
@app.websocket("/ws/meeting/{meeting_id}/stt")
async def stt_websocket(websocket: WebSocket, meeting_id: str):
    """
    Real-time STT streaming via WebSocket
    
    Flow:
    1. Browser connects
    2. Browser sends audio chunks (PCM 16kHz)
    3. Server forwards to Deepgram
    4. Deepgram returns transcript
    5. Server broadcasts to all participants in meeting
    """
    await websocket.accept()
    
    try:
        # 1. Verify user has access to meeting
        user = await get_ws_user(websocket, db)
        meeting = get_meeting_by_id(db, meeting_id)
        if not require_meeting_room_access(user, meeting, db):
            await websocket.close(code=403, reason="Forbidden")
            return
        
        # 2. Setup Deepgram streaming
        deepgram_client = DeepgramClient(api_key=config.deepgram_api_key)
        
        async with deepgram_client.live.v1.listen(
            options={
                "model": "nova-3",
                "language": "vi",
                "smart_format": True,
                "interim_results": True
            }
        ) as dg_connection:
            
            # 3. Receive audio chunks from browser
            while True:
                data = await websocket.receive_bytes()
                
                # 4. Send to Deepgram
                await dg_connection.send(data)
                
                # 5. Receive transcript
                transcript = dg_connection.recv()
                
                # 6. Process with PhoBERT
                processed = await phobert_processor.process(transcript)
                
                # 7. Broadcast to other participants
                await meeting_room_manager.broadcast_to_meeting(
                    meeting_id,
                    {
                        "type": "transcript",
                        "data": processed,
                        "speaker": user.id
                    }
                )
                
    except WebSocketDisconnect:
        logger.info(f"Client disconnected from {meeting_id}")
    except Exception as e:
        logger.error(f"WebSocket error: {e}")
        await websocket.close(code=1011, reason="Server error")
```

---

## Frontend Architecture

### 1. Project Structure

```
frontend/src/
├── pages/                   # Route pages (React Router)
│   ├── Landing.tsx
│   ├── Login.tsx
│   ├── Dashboard.tsx
│   ├── MeetingList.tsx
│   ├── MeetingDetail.tsx
│   ├── MeetingRoom.tsx      # Live meeting with STT
│   ├── ActionItems.tsx
│   ├── Notifications.tsx
│   ├── group/
│   │   ├── GroupDetail.tsx
│   │   └── CreateGroup.tsx
│   ├── org/
│   │   └── OrgAdminConsole.tsx
│   ├── admin/
│   │   ├── SystemAdminConsole.tsx
│   │   └── components/
│   │       ├── AdminOrganizations.tsx
│   │       ├── AdminUsers.tsx
│   │       ├── AdminAIServices.tsx
│   │       └── ...
│   └── profile/
│       └── Profile.tsx
│
├── features/                # Feature-level logic (state + hooks)
│   ├── admin/
│   │   ├── components/
│   │   └── stores/adminStore.ts
│   ├── calendar/
│   │   └── CalendarView.tsx
│   ├── meeting/
│   │   ├── components/MeetingCard.tsx
│   │   └── ...
│   └── meeting-room/
│       └── utils.ts
│
├── components/              # Reusable UI components
│   ├── layout/
│   │   ├── Header.tsx
│   │   ├── Sidebar.tsx
│   │   ├── OrgSelector.tsx  # Org switcher
│   │   ├── GroupNav.tsx     # Group navigator
│   │   └── ...
│   ├── meeting/
│   │   ├── MeetingCard.tsx
│   │   ├── ScheduleMeetingModal.tsx
│   │   ├── AudioPlayer.tsx  # Wavesurfer
│   │   ├── ActionItemComposer.tsx
│   │   └── ...
│   ├── group/
│   │   ├── CreateGroupModal.tsx
│   │   ├── GroupMembersTab.tsx
│   │   ├── GroupSettingsTab.tsx
│   │   └── ...
│   ├── ui/                  # Atomic UI components
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Modal.tsx
│   │   ├── Toast.tsx
│   │   ├── Badge.tsx
│   │   └── ...
│   └── landing/             # Landing page sections
│       ├── HeroSection.tsx
│       ├── FeaturesSection.tsx
│       ├── PricingSection.tsx
│       └── ...
│
├── services/                # API calls
│   ├── api.ts               # Axios instance
│   ├── auth.ts              # Auth API
│   ├── meetings.ts          # Meeting API
│   ├── organizations.ts     # Org API
│   ├── mappers.ts           # Response mappers
│   └── ...
│
├── stores/                  # Zustand stores (client state)
│   ├── authStore.ts
│   ├── orgStore.ts          # Current org + list orgs
│   ├── groupStore.ts        # Current group + list groups
│   ├── meetingStore.ts
│   ├── uiStore.ts           # UI state (sidebar, theme)
│   └── ...
│
├── context/                 # React Context
│   └── AuthContext.tsx      # Auth context (login, session)
│
├── hooks/                   # Custom React hooks
│   ├── useAuth.ts           # Auth context hook
│   ├── useCurrentRole.ts    # Current user role
│   ├── usePermission.ts     # Permission check
│   ├── useAudioRecorder.ts  # Audio recording
│   └── ...
│
├── types/                   # TypeScript interfaces
│   └── index.ts             # All types + interfaces
│
├── data/                    # Mock data + constants
│   ├── groups.ts
│   ├── orgs.ts
│   ├── users.ts
│   ├── roleMappings.ts
│   ├── roles.ts
│   └── ...
│
├── constants/               # App constants
│   └── sttCapabilities.ts   # STT config
│
├── utils/                   # Helper functions
│   ├── userSync.ts
│   └── ...
│
├── lib/                     # Utility functions
│   └── ...
│
├── assets/                  # Images, fonts, SVGs
│   └── ...
│
├── App.tsx                  # Root component
├── main.tsx                 # Entry point
└── tests/                   # Vitest suites
    └── ...
```

### 2. React Router Setup

```tsx
// App.tsx
<BrowserRouter>
  <Routes>
    {/* Public routes */}
    <Route path="/" element={<Landing />} />
    <Route path="/login" element={<Login />} />
    <Route path="/register" element={<Register />} />
    
    {/* Protected routes */}
    <Route element={<Layout />}>  {/* Main layout with Sidebar */}
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/meetings" element={<MeetingList />} />
      <Route path="/meetings/:meetingId" element={<MeetingDetail />} />
      <Route path="/meetings/:meetingId/room" element={<MeetingRoom />} />
      <Route path="/action-items" element={<ActionItems />} />
      <Route path="/notifications" element={<Notifications />} />
      
      {/* Group routes */}
      <Route path="/groups/:groupId" element={<GroupDetail />} />
      
      {/* Org routes */}
      <Route path="/org/admin" element={<OrgAdminConsole />} />
    </Route>
    
    {/* Admin routes */}
    <Route element={<AdminLayout />}>
      <Route path="/admin" element={<SystemAdminConsole />} />
      <Route path="/admin/users" element={<AdminUsers />} />
    </Route>
    
    {/* Error routes */}
    <Route path="/404" element={<NotFound />} />
    <Route path="/403" element={<Forbidden />} />
  </Routes>
</BrowserRouter>
```

### 3. State Management

#### Zustand Stores

```typescript
// File: frontend/src/stores/orgStore.ts
import { create } from 'zustand'

interface OrgStore {
  currentOrg: string | null
  organizations: Organization[]
  loading: boolean
  
  // Actions
  setCurrentOrg: (orgId: string) => void
  setOrganizations: (orgs: Organization[]) => void
  addOrganization: (org: Organization) => void
  removeOrganization: (orgId: string) => void
}

export const useOrgStore = create<OrgStore>((set) => ({
  currentOrg: null,
  organizations: [],
  loading: false,
  
  setCurrentOrg: (orgId) => set({ currentOrg: orgId }),
  setOrganizations: (orgs) => set({ organizations: orgs }),
  addOrganization: (org) => set((state) => ({
    organizations: [...state.organizations, org]
  })),
  removeOrganization: (orgId) => set((state) => ({
    organizations: state.organizations.filter(o => o.id !== orgId)
  }))
}))
```

#### React Query for Server State

```typescript
// File: frontend/src/services/meetings.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import api from './api'

export const useMeetings = () => {
  return useQuery({
    queryKey: ['meetings'],
    queryFn: async () => {
      const { data } = await api.get('/api/meetings')
      return data
    },
    staleTime: 30000  // 30 seconds
  })
}

export const useCreateMeeting = () => {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: async (meetingData) => {
      const { data } = await api.post('/api/meetings', meetingData)
      return data
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['meetings'] })
    }
  })
}
```

#### AuthContext for Authentication

```typescript
// File: frontend/src/context/AuthContext.tsx
import { createContext, useCallback, useState } from 'react'

interface AuthContextType {
  user: User | null
  session: Session | null
  login: (username: string, password: string) => Promise<User>
  logout: () => void
  isAuthenticated: boolean
  hasPermission: (permission: string) => boolean
}

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null)
  const [session, setSession] = useState<Session | null>(null)
  
  const login = useCallback(async (username: string, password: string) => {
    const response = await api.post('/api/auth/login', { username, password })
    const { token, user } = response.data
    
    // Save to localStorage
    localStorage.setItem('session', JSON.stringify({ token }))
    localStorage.setItem('user', JSON.stringify(user))
    
    setUser(user)
    setSession({ token })
    return user
  }, [])
  
  const logout = useCallback(() => {
    localStorage.removeItem('session')
    localStorage.removeItem('user')
    setUser(null)
    setSession(null)
  }, [])
  
  return (
    <AuthContext.Provider value={{ user, session, login, logout, ... }}>
      {children}
    </AuthContext.Provider>
  )
}
```

### 4. API Integration (Axios)

```typescript
// File: frontend/src/services/api.ts
import axios from 'axios'

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8000',
  headers: {
    'Content-Type': 'application/json'
  }
})

// Add JWT token to all requests
api.interceptors.request.use((config) => {
  const session = JSON.parse(localStorage.getItem('session') || '{}')
  if (session.token) {
    config.headers.Authorization = `Bearer ${session.token}`
  }
  return config
})

// Handle 401 (redirect to login)
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('session')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)

export default api
```

### 5. Component Example: OrgSelector

```tsx
// File: frontend/src/components/layout/OrgSelector.tsx
import { useOrgStore } from '../../stores'
import { useMemo } from 'react'

export const OrgSelector: React.FC = () => {
  const { currentOrg, organizations, setCurrentOrg } = useOrgStore()
  const selectedOrg = useMemo(
    () => organizations.find(o => o.id === currentOrg),
    [currentOrg, organizations]
  )
  
  return (
    <div className="flex items-center gap-2">
      <select
        value={currentOrg || ''}
        onChange={(e) => setCurrentOrg(e.target.value)}
        className="px-2 py-1 border rounded"
      >
        {organizations.map(org => (
          <option key={org.id} value={org.id}>
            {org.name}
          </option>
        ))}
      </select>
      <span className="text-sm text-gray-500">
        {selectedOrg?.name || 'Select organization'}
      </span>
    </div>
  )
}
```

---

## Database Schema

### 24 Entities Overview

```sql
-- Users & Authentication (6 tables)
users                      # User accounts (system-admin, member)
user_organizations         # M2M (user → org, with role)
password_reset_otps        # OTP for password reset
user_invitations           # Invitations to organizations
group_memberships          # M2M (user → group, with role)

-- Organizations & Groups (2 tables)
organizations              # Company/workspace
groups                     # Teams within organization

-- Meetings & Content (7 tables)
meetings                   # Meeting records
meeting_participants       # M2M (user → meeting)
transcripts                # Meeting transcripts (raw + processed)
transcript_segments        # Semantic search segments
meeting_messages           # Chat messages in meeting
action_items               # Tasks from meeting
action_item_assignees      # M2M (action_item → user)

-- AI & Processing (5 tables)
ai_data                    # Summary, processed data
meeting_settings           # Meeting-specific config
notification_preferences   # User notification settings
system_config              # Global system settings
audit_logs                 # Change tracking

-- Files & Jobs (3 tables)
export_files               # Generated PDF/DOCX/CSV
upload_jobs                # STT job queue
notifications              # Notification records
```

### Entity Relationships

```
User (1) ←──→ (M) UserOrganization ←──→ (1) Organization
  │
  ├─→ GroupMembership ←──→ Group ←──→ Organization
  ├─→ Meeting (created_by)
  ├─→ MeetingParticipant ←──→ Meeting
  ├─→ ActionItem (assigned_to, created_by)
  └─→ Notification

Meeting (1) ─→ (M) Transcript ─→ (M) TranscriptSegment
  ├─→ MeetingMessage
  ├─→ ActionItem
  ├─→ AIData
  └─→ ExportFile
```

### Foreign Key Cascade

```
User DELETE
  ├─ Cascade delete: UserOrganization, GroupMembership
  ├─ Cascade delete: MeetingParticipant
  └─ Cascade delete: ActionItem (created_by)

Organization DELETE
  ├─ Cascade delete: UserOrganization
  ├─ Cascade delete: Group
  └─ Cascade delete: Meeting

Group DELETE
  ├─ Cascade delete: GroupMembership
  └─ Cascade delete: Meeting

Meeting DELETE
  ├─ Cascade delete: Transcript, TranscriptSegment
  ├─ Cascade delete: MeetingParticipant
  ├─ Cascade delete: MeetingMessage
  ├─ Cascade delete: ActionItem
  ├─ Cascade delete: AIData
  └─ Cascade delete: ExportFile
```

---

## API Design

### RESTful Conventions

```
# Organizations
GET    /api/organizations              # List orgs (paginated)
POST   /api/organizations              # Create org
GET    /api/organizations/{org_id}     # Get org detail
PUT    /api/organizations/{org_id}     # Update org
DELETE /api/organizations/{org_id}     # Delete org

# Groups
GET    /api/organizations/{org_id}/groups              # List groups in org
POST   /api/organizations/{org_id}/groups              # Create group
GET    /api/organizations/{org_id}/groups/{group_id}   # Get group detail
PUT    /api/organizations/{org_id}/groups/{group_id}   # Update group
DELETE /api/organizations/{org_id}/groups/{group_id}   # Delete group

# Meetings
GET    /api/meetings                   # List meetings
POST   /api/meetings                   # Create meeting
GET    /api/meetings/{meeting_id}      # Get meeting detail
PUT    /api/meetings/{meeting_id}      # Update meeting
DELETE /api/meetings/{meeting_id}      # Delete meeting

# STT & Upload
POST   /api/stt/upload                 # Upload audio file
WS     /ws/meeting/{meeting_id}/stt    # WebSocket STT streaming
GET    /api/meetings/{meeting_id}/transcript  # Get transcript

# Admin
GET    /api/admin/users                # List users (system-admin only)
GET    /api/admin/organizations        # List all orgs (system-admin only)
POST   /api/admin/organizations/{org_id}/approve  # Approve org

# Analytics
GET    /api/analytics/costs            # Cost tracking
GET    /api/analytics/meetings         # Meeting statistics
GET    /api/analytics/users            # User statistics
```

### Request/Response Format

```typescript
// Request
POST /api/meetings
Content-Type: application/json
Authorization: Bearer <JWT_TOKEN>

{
  "title": "Team Standup",
  "description": "Daily meeting",
  "groupId": "group-123",
  "startTime": "2026-05-25T09:00:00Z",
  "endTime": "2026-05-25T09:30:00Z",
  "participantIds": ["user-1", "user-2"],
  "recordingEnabled": true,
  "aiSummaryEnabled": true
}

// Response (201 Created)
{
  "id": "meeting-abc123",
  "title": "Team Standup",
  "groupId": "group-123",
  "createdBy": "user-1",
  "status": "scheduled",
  "createdAt": "2026-05-25T08:30:00Z",
  "updatedAt": "2026-05-25T08:30:00Z"
}
```

### Error Handling

```typescript
// Error Response Format
{
  "detail": "Organization not found",
  "status": 404,
  "type": "not_found"
}

// Common Status Codes
200 OK              # Successful request
201 Created         # Resource created
400 Bad Request     # Invalid input
401 Unauthorized    # Missing/invalid token
403 Forbidden       # Permission denied (wrong role/scope)
404 Not Found       # Resource not found
409 Conflict        # Duplicate/conflict
422 Unprocessable   # Validation error
429 Too Many Req    # Rate limited
500 Server Error    # Unexpected error
```

---

## Data Flow

### Authentication Flow

```
1. User enters credentials (email, password)
   ↓
2. Frontend POST /api/auth/login
   ├─ Backend: Validate credentials
   ├─ Backend: Hash password check
   ├─ Backend: Generate JWT token
   └─ Backend: Return token + user data
   ↓
3. Frontend: Save token to localStorage
   ├─ localStorage.setItem('session', {token})
   └─ localStorage.setItem('user', user)
   ↓
4. Frontend: Add token to Authorization header
   └─ api.interceptors.request → "Bearer {token}"
   ↓
5. All subsequent requests include JWT token
```

### Meeting Creation & STT Flow

```
1. User creates meeting (title, time, participants)
   ↓
2. Frontend POST /api/meetings
   └─ Backend: Create Meeting record + MeetingParticipant links
   ↓
3. During meeting: User uploads audio file
   ↓
4. Frontend POST /api/stt/upload (multipart/form-data)
   ├─ Backend: Save audio file to storage
   ├─ Backend: Create UploadJob in queue
   └─ Backend: Return job ID
   ↓
5. Backend processes audio asynchronously:
   ├─ Call Deepgram STT (Nova-3) → Transcript
   ├─ Call PhoBERT processor → Tone, entities
   ├─ Call BARTpho enhancer → Polish Vietnamese
   ├─ Call Groq LLM → Context correction
   └─ Save to Transcript table
   ↓
6. Frontend polls GET /api/jobs/{job_id}/status
   ├─ Waiting → Processing → Complete
   └─ When complete: Display transcript
```

### WebSocket Real-time STT Flow

```
1. User joins meeting room
   ↓
2. Frontend establishes WebSocket
   └─ WS /ws/meeting/{meeting_id}/stt
   ↓
3. Backend accepts connection + validates user
   ├─ Check JWT token
   ├─ Check meeting access (role + group)
   └─ If valid: Accept; else: Close (403)
   ↓
4. User starts recording audio
   ↓
5. Frontend captures audio chunks (PCM 16kHz)
   ├─ Record for 100ms intervals
   └─ Send via WebSocket
   ↓
6. Backend receives audio chunks
   ├─ Forward to Deepgram (streaming)
   └─ Receive interim/final transcript
   ↓
7. Backend processes transcript
   ├─ PhoBERT tone analysis
   ├─ BARTpho enhancement
   └─ Groq context correction
   ↓
8. Backend broadcasts to all participants
   └─ WS message: {type: "transcript", data: {...}, speaker: user_id}
   ↓
9. All connected clients receive & display real-time transcript
```

### AI Summary Generation Flow

```
1. Meeting ends
   ↓
2. Backend triggers CrewAI orchestrator
   ├─ 15 agents process meeting data:
   │  ├─ DocumentProcessor
   │  ├─ KnowledgeExtractor
   │  ├─ ActionItemIdentifier
   │  ├─ StakeholderAnalyzer
   │  └─ ... (12 more agents)
   ↓
3. Each agent makes LLM calls (Groq)
   ├─ Agent 1 → Extract key topics
   ├─ Agent 2 → Identify action items
   ├─ Agent 3 → Generate summary
   └─ ... (12 more)
   ↓
4. CrewAI orchestrator merges results
   ↓
5. Backend saves to AIData table
   ├─ summary (text)
   ├─ processed_data (JSON)
   ├─ sentiment (positive/negative/neutral)
   └─ cost_data (token count, latency)
   ↓
6. Frontend fetches & displays summary
   └─ GET /api/meetings/{meeting_id}/ai-data
```

---

## Deployment Architecture

### Docker Compose (Local Development)

```yaml
services:
  backend:
    image: python:3.11-slim
    ports: ["8000:8000"]
    volumes: ["./backend:/app"]
    env_file: .env
    
  frontend:
    image: node:20-alpine
    ports: ["80:5173"]
    volumes: ["./frontend:/app"]
    
  db:
    image: mysql:8.0
    ports: ["3306:3306"]
    environment:
      MYSQL_DATABASE: convia
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
    volumes: ["mysql-data:/var/lib/mysql"]
    
  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
    volumes: ["redis-data:/data"]
    
  phpmyadmin:
    image: phpmyadmin:latest
    ports: ["5050:80"]
    environment:
      PMA_HOST: db
      PMA_USER: root
    depends_on: [db]
```

### Production Deployment (Vercel/Cloud)

```
┌─────────────────────────────────────────────────┐
│            CDN (Edge Network)                     │
│  - Static assets (JS, CSS, images)              │
│  - Cache strategy: 1 year for hashed files      │
└────────────┬────────────────────────────────────┘
             ↓
┌─────────────────────────────────────────────────┐
│         Frontend (Vercel/AWS)                    │
│  - React 19 + Vite (SSR optional)               │
│  - Auto-scaling: 0-100 instances                │
└────────────┬────────────────────────────────────┘
             ↓
     ┌───────┴────────┐
     ↓                ↓
┌──────────┐    ┌──────────────┐
│ API GW   │    │  WebSocket   │
│(FastAPI) │    │  Server      │
│ Auto:0-50│    │ (Sticky sess)│
└────┬─────┘    └──────┬───────┘
     │                 │
     └────────┬────────┘
              ↓
      ┌──────────────────┐
      │  Load Balancer   │
      │  (Round-robin)   │
      └──────────────────┘
              ↓
┌─────────────────────────────────────────────────┐
│         Database Tier (AWS RDS)                  │
│  - MySQL 8.0 (Multi-AZ, automated backups)     │
│  - Replica for read-heavy queries               │
└─────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────┐
│         Cache Tier (Redis Elasticache)          │
│  - Session storage                              │
│  - Job queue (Phase 3)                         │
│  - Rate limit counters                          │
└─────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────┐
│         Storage (S3 / Blob)                      │
│  - Audio files (.wav, .mp3)                     │
│  - Export files (PDF, DOCX, CSV)                │
│  - User avatars                                 │
└─────────────────────────────────────────────────┘
```

---

## Summary

| Layer | Technology | Key Files |
|-------|-----------|-----------|
| **UI** | React 19 + TypeScript | `/frontend/src/pages/`, `/components/` |
| **State** | Zustand + React Query | `/stores/`, `/services/` |
| **API** | Axios | `/services/api.ts` |
| **Routing** | React Router v7 | `App.tsx` |
| **Backend** | FastAPI 1.0 | `/backend/src/api/app.py` |
| **Routes** | 13 Routers | `/backend/src/api/routes/` |
| **Logic** | Core Operations | `/backend/src/api/core/` |
| **CRUD** | SQLAlchemy | `/backend/src/api/crud/` |
| **Database** | MySQL 8.0 (SQLite dev) | `/backend/src/api/models.py` |
| **Auth** | JWT + Dependencies | `/backend/src/api/auth.py` |
| **AI** | Deepgram + Groq + PhoBERT | `/backend/src/providers/` |
| **WebSocket** | FastAPI + asyncio | `/backend/src/api/routes/stt.py` |
| **DevOps** | Docker + Docker Compose | `docker-compose.yml` |
