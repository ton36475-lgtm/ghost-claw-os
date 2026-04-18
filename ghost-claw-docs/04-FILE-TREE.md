# Ghost Claw OS - File Tree & Monorepo Structure

**Version:** 1.0  
**Status:** Production-Ready  
**Author:** Manus AI  
**Date:** April 2026

---

## Monorepo Root Structure

```
ghost-claw-os/
├── apps/                           # Application packages
│   ├── web/                        # Next.js frontend
│   │   ├── app/                    # App router (Next.js 15)
│   │   │   ├── (auth)/             # Authentication pages
│   │   │   │   ├── login/
│   │   │   │   ├── signup/
│   │   │   │   └── callback/
│   │   │   ├── (dashboard)/        # Dashboard pages
│   │   │   │   ├── page.tsx        # Dashboard home
│   │   │   │   ├── projects/
│   │   │   │   └── settings/
│   │   │   ├── modules/            # Feature modules
│   │   │   │   ├── story-engine/
│   │   │   │   │   ├── page.tsx
│   │   │   │   │   ├── components/
│   │   │   │   │   └── hooks/
│   │   │   │   ├── autocut-studio/
│   │   │   │   ├── asset-library/
│   │   │   │   ├── prompt-lab/
│   │   │   │   ├── render-queue/
│   │   │   │   ├── review-gate/
│   │   │   │   ├── release-center/
│   │   │   │   └── watch-center/
│   │   │   ├── layout.tsx          # Root layout
│   │   │   └── page.tsx            # Home page
│   │   ├── components/             # Reusable components
│   │   │   ├── ui/                 # Base UI components
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Input.tsx
│   │   │   │   ├── Select.tsx
│   │   │   │   ├── Card.tsx
│   │   │   │   ├── Modal.tsx
│   │   │   │   ├── Tabs.tsx
│   │   │   │   └── Badge.tsx
│   │   │   ├── modules/            # Module-specific components
│   │   │   │   ├── StoryCard.tsx
│   │   │   │   ├── AssetGrid.tsx
│   │   │   │   ├── RenderJobList.tsx
│   │   │   │   └── AnalyticsChart.tsx
│   │   │   ├── layout/             # Layout components
│   │   │   │   ├── Header.tsx
│   │   │   │   ├── Sidebar.tsx
│   │   │   │   └── Footer.tsx
│   │   │   └── common/             # Common components
│   │   │       ├── Loading.tsx
│   │   │       ├── Error.tsx
│   │   │       └── Notification.tsx
│   │   ├── hooks/                  # Custom React hooks
│   │   │   ├── useStoryGeneration.ts
│   │   │   ├── useVideoProcessing.ts
│   │   │   ├── useAssetSearch.ts
│   │   │   ├── useReleaseManagement.ts
│   │   │   ├── useAnalytics.ts
│   │   │   └── useAuth.ts
│   │   ├── lib/                    # Utility functions
│   │   │   ├── api.ts              # API client
│   │   │   ├── auth.ts             # Auth utilities
│   │   │   ├── constants.ts        # Constants
│   │   │   └── utils.ts            # Helper functions
│   │   ├── styles/                 # Global styles
│   │   │   ├── globals.css
│   │   │   ├── variables.css
│   │   │   └── tailwind.css
│   │   ├── public/                 # Static assets
│   │   │   ├── images/
│   │   │   ├── icons/
│   │   │   └── fonts/
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── tailwind.config.ts
│   │   ├── next.config.ts
│   │   └── .env.example
│   │
│   └── docs/                       # Documentation site (Docusaurus)
│       ├── docs/                   # Documentation content
│       │   ├── intro.md
│       │   ├── getting-started/
│       │   ├── guides/
│       │   └── api/
│       ├── src/
│       ├── package.json
│       └── docusaurus.config.ts
│
├── packages/                       # Shared packages
│   ├── api-fastapi/                # Python FastAPI backend
│   │   ├── app/
│   │   │   ├── main.py             # FastAPI app entry
│   │   │   ├── config.py           # Configuration
│   │   │   ├── database.py         # Database setup
│   │   │   ├── models/             # SQLAlchemy models
│   │   │   │   ├── user.py
│   │   │   │   ├── project.py
│   │   │   │   ├── story.py
│   │   │   │   ├── review.py
│   │   │   │   └── asset.py
│   │   │   ├── schemas/            # Pydantic schemas
│   │   │   │   ├── story.py
│   │   │   │   ├── review.py
│   │   │   │   └── asset.py
│   │   │   ├── routers/            # API routes
│   │   │   │   ├── stories.py
│   │   │   │   ├── videos.py
│   │   │   │   ├── reviews.py
│   │   │   │   ├── assets.py
│   │   │   │   └── prompts.py
│   │   │   ├── services/           # Business logic
│   │   │   │   ├── story_service.py
│   │   │   │   ├── video_service.py
│   │   │   │   ├── review_service.py
│   │   │   │   ├── asset_service.py
│   │   │   │   └── prompt_service.py
│   │   │   ├── workers/            # Background workers
│   │   │   │   ├── story_worker.py
│   │   │   │   ├── video_worker.py
│   │   │   │   ├── render_worker.py
│   │   │   │   └── review_worker.py
│   │   │   ├── utils/              # Utilities
│   │   │   │   ├── llm.py
│   │   │   │   ├── ffmpeg.py
│   │   │   │   ├── google_drive.py
│   │   │   │   └── validators.py
│   │   │   └── middleware/         # Middleware
│   │   │       ├── auth.py
│   │   │       └── logging.py
│   │   ├── tests/                  # Unit tests
│   │   │   ├── test_story_service.py
│   │   │   ├── test_video_service.py
│   │   │   └── test_review_service.py
│   │   ├── migrations/             # Alembic migrations
│   │   │   └── versions/
│   │   ├── requirements.txt        # Python dependencies
│   │   ├── pyproject.toml          # Poetry config
│   │   ├── Dockerfile
│   │   └── .env.example
│   │
│   ├── api-nestjs/                 # Node.js NestJS backend
│   │   ├── src/
│   │   │   ├── main.ts             # NestJS app entry
│   │   │   ├── app.module.ts       # Root module
│   │   │   ├── modules/
│   │   │   │   ├── users/
│   │   │   │   │   ├── users.module.ts
│   │   │   │   │   ├── users.service.ts
│   │   │   │   │   ├── users.controller.ts
│   │   │   │   │   └── dto/
│   │   │   │   ├── projects/
│   │   │   │   ├── releases/
│   │   │   │   ├── analytics/
│   │   │   │   ├── notifications/
│   │   │   │   └── incidents/
│   │   │   ├── common/
│   │   │   │   ├── decorators/
│   │   │   │   ├── filters/
│   │   │   │   ├── guards/
│   │   │   │   ├── interceptors/
│   │   │   │   └── pipes/
│   │   │   ├── database/
│   │   │   │   ├── database.module.ts
│   │   │   │   └── entities/
│   │   │   ├── config/
│   │   │   │   ├── database.config.ts
│   │   │   │   ├── auth.config.ts
│   │   │   │   └── queue.config.ts
│   │   │   └── queue/
│   │   │       ├── queue.module.ts
│   │   │       ├── processors/
│   │   │       └── jobs/
│   │   ├── test/
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── Dockerfile
│   │   └── .env.example
│   │
│   ├── shared/                     # Shared types & utilities
│   │   ├── src/
│   │   │   ├── types/              # TypeScript types
│   │   │   │   ├── story.ts
│   │   │   │   ├── project.ts
│   │   │   │   ├── user.ts
│   │   │   │   ├── api.ts
│   │   │   │   └── common.ts
│   │   │   ├── constants/          # Shared constants
│   │   │   │   ├── platforms.ts
│   │   │   │   ├── status.ts
│   │   │   │   └── roles.ts
│   │   │   ├── utils/              # Shared utilities
│   │   │   │   ├── validation.ts
│   │   │   │   ├── formatting.ts
│   │   │   │   └── date.ts
│   │   │   └── index.ts            # Barrel export
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── ml-workers/                 # ML processing workers
│       ├── src/
│       │   ├── main.py
│       │   ├── workers/
│       │   │   ├── scene_detection.py
│       │   │   ├── transcription.py
│       │   │   ├── semantic_ranking.py
│       │   │   └── key_moment_extraction.py
│       │   ├── models/
│       │   │   ├── scene_detector.py
│       │   │   ├── transcriber.py
│       │   │   └── ranker.py
│       │   └── utils/
│       │       ├── video.py
│       │       ├── audio.py
│       │       └── ml.py
│       ├── requirements.txt
│       ├── Dockerfile
│       └── .env.example
│
├── infrastructure/                 # Infrastructure as Code
│   ├── docker/
│   │   ├── Dockerfile.fastapi
│   │   ├── Dockerfile.nestjs
│   │   ├── Dockerfile.ml-workers
│   │   └── docker-compose.yml
│   ├── k8s/                        # Kubernetes manifests
│   │   ├── namespace.yaml
│   │   ├── postgres.yaml
│   │   ├── redis.yaml
│   │   ├── fastapi.yaml
│   │   ├── nestjs.yaml
│   │   ├── ingress.yaml
│   │   └── configmap.yaml
│   ├── terraform/                  # Terraform IaC
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── ecs.tf
│   │   ├── rds.tf
│   │   ├── elasticache.tf
│   │   ├── s3.tf
│   │   └── networking.tf
│   └── scripts/
│       ├── deploy.sh
│       ├── rollback.sh
│       ├── backup.sh
│       └── restore.sh
│
├── docs/                           # Documentation
│   ├── 01-PRODUCT-SUMMARY.md
│   ├── 02-PRD.md
│   ├── 03-SYSTEM-ARCHITECTURE.md
│   ├── 04-FILE-TREE.md
│   ├── 06-API-ROUTES.md
│   ├── 07-DATABASE-SCHEMA.md
│   ├── 08-QUEUE-WORKER-DESIGN.md
│   ├── 09-LONG-FORM-AUTOCUT-LOGIC.md
│   ├── 10-ONE-CLICK-STORY-ENGINE-LOGIC.md
│   ├── 11-ASSET-MEMORY-LOGIC.md
│   ├── 12-REVIEW-RELEASE-WATCH-LOGIC.md
│   ├── 13-INITIAL-BUILD-PLAN.md
│   ├── 14-NEXT-STEPS.md
│   └── 15-UI-SCREENS.md
│
├── .github/                        # GitHub configuration
│   ├── workflows/
│   │   ├── ci.yml                  # CI pipeline
│   │   ├── deploy.yml              # Deployment pipeline
│   │   └── security.yml            # Security scanning
│   └── ISSUE_TEMPLATE/
│
├── package.json                    # Root package.json (pnpm)
├── pnpm-workspace.yaml             # Workspace configuration
├── turbo.json                      # Turborepo configuration
├── docker-compose.yml              # Local development environment
├── .env.example                    # Environment variables template
├── .gitignore
├── README.md                       # Getting started guide
├── LICENSE
└── CONTRIBUTING.md
```

---

## Key Files & Directories

### Frontend (Next.js)

| Path | Purpose |
|------|---------|
| `apps/web/app/` | App router pages and layouts |
| `apps/web/components/` | Reusable React components |
| `apps/web/hooks/` | Custom React hooks |
| `apps/web/lib/api.ts` | API client for backend communication |
| `apps/web/styles/` | Global CSS and Tailwind configuration |
| `apps/web/public/` | Static assets (images, icons, fonts) |

### Backend (FastAPI)

| Path | Purpose |
|------|---------|
| `packages/api-fastapi/app/main.py` | FastAPI application entry point |
| `packages/api-fastapi/app/models/` | SQLAlchemy ORM models |
| `packages/api-fastapi/app/schemas/` | Pydantic request/response schemas |
| `packages/api-fastapi/app/routers/` | API route definitions |
| `packages/api-fastapi/app/services/` | Business logic layer |
| `packages/api-fastapi/app/workers/` | Background job workers |
| `packages/api-fastapi/app/utils/` | Utility functions (LLM, FFmpeg, etc.) |

### Backend (NestJS)

| Path | Purpose |
|------|---------|
| `packages/api-nestjs/src/main.ts` | NestJS application entry point |
| `packages/api-nestjs/src/modules/` | Feature modules (users, projects, etc.) |
| `packages/api-nestjs/src/common/` | Shared middleware, guards, decorators |
| `packages/api-nestjs/src/database/` | Database configuration and entities |
| `packages/api-nestjs/src/queue/` | BullMQ queue configuration |

### Shared

| Path | Purpose |
|------|---------|
| `packages/shared/src/types/` | TypeScript type definitions |
| `packages/shared/src/constants/` | Shared constants |
| `packages/shared/src/utils/` | Shared utility functions |

### Infrastructure

| Path | Purpose |
|------|---------|
| `infrastructure/docker/` | Docker configurations |
| `infrastructure/k8s/` | Kubernetes manifests |
| `infrastructure/terraform/` | Infrastructure as Code |
| `infrastructure/scripts/` | Deployment and maintenance scripts |

### Documentation

| Path | Purpose |
|------|---------|
| `docs/01-PRODUCT-SUMMARY.md` | Product overview and value proposition |
| `docs/02-PRD.md` | Functional and non-functional requirements |
| `docs/03-SYSTEM-ARCHITECTURE.md` | System design and architecture |
| `docs/04-FILE-TREE.md` | This file - directory structure |
| `docs/06-API-ROUTES.md` | API endpoint documentation |
| `docs/07-DATABASE-SCHEMA.md` | Database schema and relationships |
| `docs/08-QUEUE-WORKER-DESIGN.md` | Job queue architecture |
| `docs/09-LONG-FORM-AUTOCUT-LOGIC.md` | Long-form video processing algorithm |
| `docs/10-ONE-CLICK-STORY-ENGINE-LOGIC.md` | Story generation algorithm |
| `docs/11-ASSET-MEMORY-LOGIC.md` | Asset management system |
| `docs/12-REVIEW-RELEASE-WATCH-LOGIC.md` | Review and release workflows |
| `docs/13-INITIAL-BUILD-PLAN.md` | Development roadmap |
| `docs/14-NEXT-STEPS.md` | Implementation guide |
| `docs/15-UI-SCREENS.md` | UI/UX design specifications |

---

## Configuration Files

### Root Configuration

**package.json:**
- Workspace definition
- Shared scripts (dev, build, test, lint)
- Root dependencies

**pnpm-workspace.yaml:**
- Workspace packages definition
- Shared dependency resolution

**turbo.json:**
- Build task orchestration
- Caching configuration
- Pipeline definition

**docker-compose.yml:**
- Local development environment
- PostgreSQL, Redis, MinIO, Elasticsearch
- Service networking

**.env.example:**
- Template for environment variables
- All required configuration keys

---

## Build & Development Workflow

### Development

```bash
# Install dependencies
pnpm install

# Start all services
pnpm dev

# Services available at:
# - Frontend: http://localhost:3000
# - FastAPI: http://localhost:8000
# - NestJS: http://localhost:3001
# - Docs: http://localhost:3002
```

### Building

```bash
# Build all packages
pnpm build

# Build specific package
pnpm build --filter=@ghost-claw-os/web

# Build with Turborepo
turbo build
```

### Testing

```bash
# Run all tests
pnpm test

# Run tests for specific package
pnpm test --filter=@ghost-claw-os/api-fastapi

# Run with coverage
pnpm test:coverage
```

---

**Document Version:** 1.0  
**Last Updated:** April 18, 2026  
**Status:** Ready for Implementation
