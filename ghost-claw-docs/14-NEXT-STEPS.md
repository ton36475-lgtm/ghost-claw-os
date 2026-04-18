# Ghost Claw OS - Next Steps & Implementation Guide

**Version:** 1.0  
**Status:** Ready for Implementation  
**Author:** Manus AI  
**Date:** April 2026

---

## Quick Start (For Developers)

### Prerequisites
```bash
# Required tools
- Node.js 18+
- Python 3.11+
- Docker & Docker Compose
- PostgreSQL 16
- Redis 7
- FFmpeg 6
- Git

# Verify installation
node --version     # v18+
python --version   # 3.11+
docker --version   # 24+
ffmpeg -version    # 6+
```

### Initial Setup (5 minutes)

```bash
# 1. Clone repository
git clone https://github.com/sirinx/ghost-claw-os.git
cd ghost-claw-os

# 2. Install dependencies
pnpm install

# 3. Set up environment
cp .env.example .env
# Edit .env with your credentials

# 4. Start infrastructure
docker-compose up -d

# 5. Initialize database
pnpm db:migrate

# 6. Start development servers
pnpm dev

# Services will be available at:
# - Frontend: http://localhost:3000
# - API (FastAPI): http://localhost:8000
# - API (NestJS): http://localhost:3001
# - Docs: http://localhost:3002
```

---

## Phase-by-Phase Implementation

### Phase 1: Foundation (Weeks 1-2)

#### Step 1.1: Clone Monorepo Template
```bash
# The monorepo structure is already created in /home/ubuntu/ghost-claw-os/
# with all configuration files:
# - package.json (root)
# - pnpm-workspace.yaml
# - turbo.json
# - docker-compose.yml
# - .env.example
# - README.md

# Verify structure
ls -la /home/ubuntu/ghost-claw-os/
```

#### Step 1.2: Set Up Development Environment
```bash
# Create .env file
cp /home/ubuntu/ghost-claw-os/.env.example /home/ubuntu/ghost-claw-os/.env

# Edit with your credentials
# - Database URL
# - Redis URL
# - Google Cloud credentials
# - OpenAI API key
# - etc.
```

#### Step 1.3: Initialize Services
```bash
# Start Docker services
cd /home/ubuntu/ghost-claw-os
docker-compose up -d

# Verify services
docker-compose ps

# Check logs
docker-compose logs -f postgres
docker-compose logs -f redis
```

#### Step 1.4: Set Up CI/CD
```bash
# GitHub Actions workflows are in .github/workflows/
# - ci.yml: Run tests on every push
# - deploy.yml: Deploy to production on main branch

# To enable:
# 1. Push code to GitHub
# 2. Enable GitHub Actions in repository settings
# 3. Add secrets (VERCEL_TOKEN, AWS_CREDENTIALS, etc.)
```

### Phase 2: Backend Infrastructure (Weeks 3-4)

#### Step 2.1: FastAPI Backend
```bash
# Navigate to FastAPI package
cd packages/api-fastapi

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run migrations
alembic upgrade head

# Start server
uvicorn app.main:app --reload --port 8000

# API docs available at http://localhost:8000/docs
```

#### Step 2.2: NestJS Backend
```bash
# Navigate to NestJS package
cd packages/api-nestjs

# Install dependencies
pnpm install

# Generate Prisma client
pnpm prisma generate

# Run migrations
pnpm prisma migrate dev

# Start server
pnpm start:dev

# API available at http://localhost:3001
```

#### Step 2.3: Database Setup
```bash
# Connect to PostgreSQL
psql -h localhost -U postgres -d ghost_claw_os

# Run schema
\i docs/07-DATABASE-SCHEMA.md

# Verify tables
\dt

# Create indexes
\i infrastructure/sql/indexes.sql
```

#### Step 2.4: Queue Setup
```bash
# Redis is already running in Docker
# Verify connection
redis-cli ping  # Should return PONG

# Monitor queue
redis-cli
> KEYS *
> HGETALL bull:video-processing:*
```

### Phase 3: Frontend Setup (Week 5)

#### Step 3.1: Next.js Application
```bash
# Navigate to web app
cd apps/web

# Install dependencies
pnpm install

# Create .env.local
cp .env.example .env.local

# Start development server
pnpm dev

# Open http://localhost:3000
```

#### Step 3.2: Project Structure
```
apps/web/
├── app/
│   ├── (auth)/          # Authentication pages
│   ├── (dashboard)/     # Dashboard pages
│   ├── modules/         # Feature modules
│   │   ├── story-engine/
│   │   ├── autocut-studio/
│   │   ├── asset-library/
│   │   ├── prompt-lab/
│   │   ├── render-queue/
│   │   ├── review-gate/
│   │   └── release-center/
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ui/              # Reusable UI components
│   ├── modules/         # Module-specific components
│   └── layout/          # Layout components
├── hooks/               # Custom React hooks
├── lib/                 # Utility functions
├── styles/              # Global styles
└── public/              # Static assets
```

#### Step 3.3: Key Components to Build
```typescript
// 1. Dashboard
apps/web/app/(dashboard)/page.tsx

// 2. Story Engine
apps/web/app/modules/story-engine/page.tsx

// 3. Asset Library
apps/web/app/modules/asset-library/page.tsx

// 4. Render Queue
apps/web/app/modules/render-queue/page.tsx

// 5. Review Gate
apps/web/app/modules/review-gate/page.tsx

// 6. Release Center
apps/web/app/modules/release-center/page.tsx
```

### Phase 4: Feature Implementation (Weeks 6-9)

#### Step 4.1: One-Click Story Engine
```bash
# Implement in order:
1. Frontend: Story input form
2. API: /api/stories/generate endpoint
3. Worker: Story generation job
4. LangGraph: Story generation logic
5. Rendering: Remotion composition
6. Output: S3 upload & metadata storage
```

#### Step 4.2: Long-form Autocut
```bash
# Implement in order:
1. Frontend: Video upload form
2. API: /api/videos/upload endpoint
3. Worker: Proxy generation job
4. Worker: Scene detection job
5. Worker: Transcription job
6. Worker: Key moment extraction
7. Output: Story extraction & assembly
```

#### Step 4.3: Asset Library
```bash
# Implement in order:
1. Google Drive integration
2. Google Sheets metadata sync
3. Asset search API
4. Asset caching
5. Frontend: Asset browser
6. Asset usage tracking
```

#### Step 4.4: Review Gate
```bash
# Implement in order:
1. AI compliance checks (FastAPI)
2. Review database schema
3. Human review API
4. Review UI (Next.js)
5. Approval workflow
6. Notification system
```

### Phase 5: Polish & Launch (Weeks 10-12)

#### Step 5.1: Testing
```bash
# Unit tests
pnpm test

# Integration tests
pnpm test:integration

# E2E tests
pnpm test:e2e

# Coverage report
pnpm test:coverage
```

#### Step 5.2: Performance Optimization
```bash
# Frontend
pnpm build
pnpm analyze  # Analyze bundle size

# Backend
# - Add database indexes
# - Implement caching
# - Optimize queries

# Infrastructure
# - Set up CDN
# - Configure auto-scaling
# - Enable compression
```

#### Step 5.3: Security Hardening
```bash
# Run security checks
pnpm audit
pnpm audit:fix

# SAST scanning
semgrep --config=p/security-audit

# Vulnerability scanning
trivy fs .

# Penetration testing
# - OWASP Top 10
# - API security
# - Authentication/Authorization
```

#### Step 5.4: Deployment
```bash
# Production build
pnpm build

# Deploy to Vercel (Frontend)
vercel --prod

# Deploy to AWS (Backend)
# - Build Docker images
# - Push to ECR
# - Deploy to ECS/EKS

# Database migration
pnpm db:migrate:prod

# Verify deployment
curl https://api.ghost-claw-os.com/health
```

---

## Sample Project: "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้"

### Project Setup

```bash
# 1. Create project in UI
POST /api/projects
{
  "name": "SME Electricity Costs",
  "description": "Educational series about unpredictable SME electricity costs",
  "campaign": "Q1-2026-SME",
  "platforms": ["youtube", "tiktok", "instagram"]
}

# 2. Create intake (for long-form video)
POST /api/intakes
{
  "project_id": "uuid",
  "video_url": "s3://bucket/sme-electricity-2h.mp4",
  "topic": "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้",
  "campaign": "Q1-2026-SME",
  "target_audience": "SME business owners",
  "platforms": ["youtube", "tiktok", "instagram"],
  "brief": "Explain why SME electricity costs are unpredictable and how to manage them"
}
```

### One-Click Story Generation

```bash
# 1. Generate stories from topic
POST /api/stories/generate
{
  "project_id": "uuid",
  "topic": "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้",
  "platforms": ["youtube", "tiktok", "instagram"],
  "style": "educational",
  "target_audience": "SME business owners",
  "cta": "Subscribe for more SME tips"
}

# Response:
{
  "generation_job_id": "uuid",
  "status": "queued",
  "estimated_time": "4 minutes"
}

# 2. Monitor progress
GET /api/jobs/uuid
{
  "status": "processing",
  "progress": 45,
  "current_stage": "Generating scenes"
}

# 3. Get generated stories
GET /api/stories?generation_job_id=uuid
{
  "stories": [
    {
      "id": "uuid",
      "title": "Why SME electricity costs vary",
      "video_url": "s3://bucket/story-1.mp4",
      "thumbnail_url": "s3://bucket/thumbnail-1.jpg",
      "duration": 8,
      "status": "draft"
    },
    // ... 7 more stories
  ]
}
```

### Long-form Autocut

```bash
# 1. Upload long-form video
POST /api/intakes
{
  "project_id": "uuid",
  "video_url": "s3://bucket/sme-electricity-2h.mp4",
  "topic": "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้"
}

# 2. Start processing
POST /api/videos/process
{
  "intake_id": "uuid",
  "processing_type": "full"  # proxy + scenes + transcription + key_moments
}

# 3. Monitor processing
GET /api/jobs/uuid
{
  "status": "processing",
  "progress": 60,
  "current_stage": "Extracting key moments"
}

# 4. Get extracted stories
GET /api/stories?intake_id=uuid
{
  "stories": [
    {
      "id": "uuid",
      "title": "Story 1: Hook",
      "video_url": "s3://bucket/story-1.mp4",
      "duration": 8,
      "status": "draft"
    },
    // ... 7 more stories
  ]
}
```

### Review & Release

```bash
# 1. Run AI review
POST /api/reviews
{
  "story_id": "uuid",
  "review_type": "ai_review"
}

# Response:
{
  "review_id": "uuid",
  "ai_score": 0.92,
  "recommendation": "auto_approved",
  "checks": {
    "brand_guidelines": { "passed": true, "score": 0.95 },
    "legal_compliance": { "passed": true, "score": 0.98 },
    "platform_policies": { "passed": true, "score": 0.88 },
    "accessibility": { "passed": true, "score": 0.92 }
  }
}

# 2. Create release
POST /api/releases
{
  "project_id": "uuid",
  "story_ids": ["uuid1", "uuid2", "uuid3", "uuid4", "uuid5", "uuid6", "uuid7", "uuid8"],
  "platforms": ["youtube", "tiktok", "instagram"],
  "scheduled_for": "2026-04-20T10:00:00Z"
}

# 3. Publish release
POST /api/releases/uuid/publish
{
  "status": "publishing"
}

# 4. Monitor analytics
GET /api/releases/uuid/analytics
{
  "platforms": {
    "youtube": {
      "views": 5000,
      "likes": 250,
      "comments": 45,
      "shares": 120,
      "engagement_rate": 0.073
    },
    "tiktok": {
      "views": 12000,
      "likes": 1200,
      "comments": 300,
      "shares": 450,
      "engagement_rate": 0.138
    },
    "instagram": {
      "views": 3000,
      "likes": 300,
      "comments": 50,
      "shares": 80,
      "engagement_rate": 0.113
    }
  }
}
```

---

## Troubleshooting Guide

### Common Issues

#### 1. Database Connection Error
```bash
# Check PostgreSQL is running
docker-compose ps postgres

# Check connection string in .env
cat .env | grep DATABASE_URL

# Test connection
psql -h localhost -U postgres -d ghost_claw_os
```

#### 2. Redis Connection Error
```bash
# Check Redis is running
docker-compose ps redis

# Test connection
redis-cli ping

# Check Redis configuration
docker-compose logs redis
```

#### 3. API Not Responding
```bash
# Check if server is running
curl http://localhost:8000/health
curl http://localhost:3001/health

# Check logs
docker-compose logs api-fastapi
docker-compose logs api-nestjs

# Restart services
docker-compose restart api-fastapi api-nestjs
```

#### 4. Video Processing Fails
```bash
# Check FFmpeg installation
ffmpeg -version

# Check job logs
GET /api/jobs/uuid/logs

# Check worker logs
docker-compose logs ml-workers

# Verify video format
ffprobe -v error -select_streams v:0 -show_entries stream=codec_type,codec_name,width,height -of default=noprint_wrappers=1 video.mp4
```

#### 5. LLM API Rate Limit
```bash
# Check API usage
curl https://api.openai.com/v1/usage

# Implement exponential backoff
# See: packages/api-nestjs/src/services/llm.service.ts

# Use caching for common prompts
# See: packages/shared/lib/cache.ts
```

---

## Performance Tuning

### Database Optimization
```sql
-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM stories WHERE project_id = 'uuid';

-- Add missing indexes
CREATE INDEX idx_stories_project_created ON stories(project_id, created_at DESC);

-- Vacuum and analyze
VACUUM ANALYZE;
```

### Redis Optimization
```bash
# Monitor Redis memory
redis-cli INFO memory

# Set max memory policy
redis-cli CONFIG SET maxmemory-policy allkeys-lru

# Monitor key expiration
redis-cli MONITOR
```

### Frontend Optimization
```bash
# Analyze bundle size
pnpm build
pnpm analyze

# Common issues:
# - Large dependencies (use tree-shaking)
# - Unoptimized images (use next/image)
# - Missing code splitting (use dynamic imports)
```

---

## Monitoring & Alerting

### Key Metrics to Monitor
```
- API response time (target: < 500ms)
- Database query time (target: < 100ms)
- Queue depth (target: < 100 jobs)
- Error rate (target: < 0.1%)
- Uptime (target: > 99.9%)
```

### Set Up Monitoring
```bash
# DataDog
export DD_API_KEY=your_api_key
export DD_SITE=datadoghq.com

# Prometheus
docker-compose up -d prometheus

# Grafana
docker-compose up -d grafana
# Access at http://localhost:3000
```

---

## Support & Resources

### Documentation
- [System Architecture](./03-SYSTEM-ARCHITECTURE.md)
- [API Documentation](./06-API-ROUTES.md)
- [Database Schema](./07-DATABASE-SCHEMA.md)
- [Queue Design](./08-QUEUE-WORKER-DESIGN.md)

### External Resources
- [Next.js Documentation](https://nextjs.org/docs)
- [FastAPI Documentation](https://fastapi.tiangolo.com)
- [NestJS Documentation](https://docs.nestjs.com)
- [Remotion Documentation](https://www.remotion.dev)
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph)

### Getting Help
- GitHub Issues: https://github.com/sirinx/ghost-claw-os/issues
- Slack Channel: #ghost-claw-os
- Email: support@sirinx.com

---

**Document Version:** 1.0  
**Last Updated:** April 18, 2026  
**Status:** Ready for Implementation
