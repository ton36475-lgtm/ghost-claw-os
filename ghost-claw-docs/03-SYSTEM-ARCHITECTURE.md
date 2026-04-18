# Ghost Claw OS - System Architecture

**Version:** 1.0  
**Status:** Production-Ready  
**Author:** Manus AI  
**Date:** April 2026

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Layer                             │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │   Next.js Web    │  │  React 19        │  │  Tailwind    │  │
│  │   Dashboard      │  │  Components      │  │  CSS         │  │
│  └──────────────────┘  └──────────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                        API Gateway                              │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Authentication | Rate Limiting | Request Validation    │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                    ↙                              ↘
┌──────────────────────────────────┐  ┌──────────────────────────────┐
│     FastAPI Backend (Python)     │  │    NestJS Backend (Node.js)  │
│  ┌────────────────────────────┐  │  │  ┌──────────────────────┐   │
│  │ Story Generation Service   │  │  │  │ Release Service      │   │
│  │ Video Processing Service   │  │  │  │ Analytics Service    │   │
│  │ Asset Management Service   │  │  │  │ Notification Service │   │
│  │ Review Service             │  │  │  │ User Service         │   │
│  │ Prompt Service             │  │  │  └──────────────────────┘   │
│  └────────────────────────────┘  │  │                              │
└──────────────────────────────────┘  └──────────────────────────────┘
        ↓                                       ↓
┌──────────────────────────────────┐  ┌──────────────────────────────┐
│       Data Layer (PostgreSQL)     │  │     Queue Layer (Redis)      │
│  ┌────────────────────────────┐  │  │  ┌──────────────────────┐   │
│  │ Users, Projects, Stories   │  │  │  │ Story Generation Job │   │
│  │ Releases, Reviews, Assets  │  │  │  │ Video Processing Job │   │
│  │ Analytics, Incidents       │  │  │  │ Render Job           │   │
│  └────────────────────────────┘  │  │  │ Review Job           │   │
│                                  │  │  │ Release Job          │   │
└──────────────────────────────────┘  │  └──────────────────────┘   │
                                       └──────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    External Services                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │ Google Drive │  │ LLM APIs     │  │ Platform APIs        │  │
│  │ Google Sheets│  │ (OpenAI,     │  │ (YouTube, TikTok,    │  │
│  │              │  │  Grok)       │  │  Instagram, LinkedIn) │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Storage & Processing                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │ S3 Storage   │  │ FFmpeg       │  │ ML Workers           │  │
│  │ (Videos,     │  │ (Rendering)  │  │ (PySceneDetect,      │  │
│  │  Assets)     │  │              │  │  Faster-Whisper)     │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Component Architecture

### Frontend Layer

**Technology Stack:**
- Next.js 15 (React framework)
- React 19 (UI library)
- Tailwind CSS (styling)
- TypeScript (type safety)
- TanStack Query (data fetching)
- Zustand (state management)

**Key Components:**
- Dashboard
- Story Engine UI
- Asset Library UI
- Render Queue UI
- Review Gate UI
- Release Center UI
- Watch Center UI

### Backend Layer

#### FastAPI (Python)

**Responsibilities:**
- Story generation (LangGraph integration)
- Video processing (FFmpeg orchestration)
- Asset management (Google Drive integration)
- Compliance review (AI checks)
- Prompt generation (LLM integration)

**Key Services:**
```python
# Story Generation Service
class StoryGenerationService:
    async def generate_stories(topic, platforms, style, target_audience, cta)
    async def get_story_generation_status(job_id)
    async def get_generated_stories(job_id)

# Video Processing Service
class VideoProcessingService:
    async def upload_video(file)
    async def process_video(video_id, processing_type)
    async def get_processing_status(job_id)
    async def get_extracted_stories(job_id)

# Asset Management Service
class AssetManagementService:
    async def sync_google_drive()
    async def search_assets(tags, usage_role, campaign)
    async def get_asset_details(asset_id)
    async def track_asset_usage(asset_id, story_id)

# Review Service
class ReviewService:
    async def run_ai_review(story_id)
    async def submit_human_review(story_id, review_data)
    async def get_review_status(story_id)
```

#### NestJS (Node.js)

**Responsibilities:**
- User management
- Project management
- Release management
- Analytics aggregation
- Notification system
- Platform integration

**Key Services:**
```typescript
// Release Service
class ReleaseService {
  async createRelease(releaseData): Promise<Release>
  async scheduleRelease(releaseId): Promise<void>
  async publishRelease(releaseId): Promise<void>
  async getReleaseStatus(releaseId): Promise<ReleaseStatus>
}

// Analytics Service
class AnalyticsService {
  async trackAnalytics(releaseId): Promise<void>
  async getAnalytics(releaseId): Promise<Analytics>
  async getPerformanceMetrics(storyId): Promise<Metrics>
}

// Notification Service
class NotificationService {
  async sendNotification(userId, message): Promise<void>
  async sendSlackAlert(message): Promise<void>
}

// User Service
class UserService {
  async createUser(userData): Promise<User>
  async updateUser(userId, userData): Promise<User>
  async getUserPermissions(userId): Promise<Permission[]>
}
```

### Queue Layer (BullMQ + Redis)

**Job Types:**

| Job Type | Service | Priority | Timeout |
|----------|---------|----------|---------|
| story-generation | FastAPI | High | 5 min |
| video-processing | FastAPI | High | 30 min |
| rendering | FastAPI | Medium | 30 min |
| review | FastAPI | Medium | 5 min |
| release | NestJS | Medium | 10 min |

**Queue Configuration:**
```typescript
const queues = {
  storyGeneration: new Queue('story-generation', { redis }),
  videoProcessing: new Queue('video-processing', { redis }),
  rendering: new Queue('rendering', { redis }),
  review: new Queue('review', { redis }),
  release: new Queue('release', { redis })
};

// Job processing
storyGeneration.process(async (job) => {
  const { topic, platforms, style } = job.data;
  return await storyGenerationService.generate(topic, platforms, style);
});
```

### Data Layer (PostgreSQL)

**Database Schema:**

The database consists of 30+ tables organized into logical domains:

**User Management:**
- users
- user_roles
- user_permissions
- team_members

**Project Management:**
- projects
- campaigns
- project_settings

**Content:**
- stories
- story_versions
- story_metadata
- scenes

**Processing:**
- intakes
- video_uploads
- processing_jobs
- job_logs

**Assets:**
- assets
- asset_metadata
- asset_usage

**Reviews:**
- reviews
- review_checks
- review_annotations

**Releases:**
- releases
- release_stories
- platform_publish_status

**Analytics:**
- analytics_events
- platform_metrics
- engagement_metrics

**Incidents:**
- incidents
- incident_logs

### Storage Layer (S3)

**Bucket Structure:**
```
s3://ghost-claw-os/
├── projects/
│   ├── {project_id}/
│   │   ├── videos/
│   │   ├── stories/
│   │   ├── thumbnails/
│   │   └── assets/
├── processing/
│   ├── proxies/
│   ├── scenes/
│   └── transcriptions/
└── backups/
```

### Processing Layer

**FFmpeg Workers:**
- Video proxy generation
- Video rendering
- Format conversion
- Thumbnail generation

**ML Workers:**
- Scene detection (PySceneDetect)
- Speech-to-text (Faster-Whisper)
- Image generation (Stable Diffusion via API)
- Video generation (Remotion)

---

## Data Flow

### One-Click Story Generation Flow

```
User Input (Topic, Platforms, Style, Target, CTA)
    ↓
Validation & Sanitization
    ↓
Queue Job: story-generation
    ↓
FastAPI Worker:
  1. Generate hook (LLM)
  2. Extract core angle (LLM)
  3. Select story pattern (LLM)
  4. Generate 6-8 scenes (LLM)
  5. Generate visual prompts (LLM)
  6. Generate voiceover script (LLM)
  7. Generate on-screen text (LLM)
  8. Generate CTA (LLM)
  9. Generate thumbnail prompt (LLM)
    ↓
Queue Job: rendering (8 jobs in parallel)
    ↓
Remotion Worker:
  1. Generate video from composition
  2. Add voiceover
  3. Add captions
  4. Add music/SFX
  5. Render final video
    ↓
Upload to S3
    ↓
Store metadata in PostgreSQL
    ↓
Return to frontend
```

### Long-form Autocut Flow

```
User Upload: 2h+ video
    ↓
Validation & Storage in S3
    ↓
Queue Job: video-processing
    ↓
FastAPI Worker:
  1. Generate proxy (FFmpeg) - 5 min
  2. Scene detection (PySceneDetect) - 5 min
  3. Transcription (Faster-Whisper) - 10 min
  4. Semantic ranking (LLM) - 5 min
  5. Key moment extraction (LLM) - 3 min
  6. Story assembly (FFmpeg) - 2 min
    ↓
Queue Job: rendering (8-12 jobs in parallel)
    ↓
Remotion Worker:
  1. Assemble scenes
  2. Add transitions
  3. Add B-roll
  4. Add captions
  5. Add music/SFX
  6. Render final video
    ↓
Upload to S3
    ↓
Store metadata in PostgreSQL
    ↓
Return to frontend
```

### Review & Release Flow

```
Story Generated
    ↓
Queue Job: review
    ↓
FastAPI Worker (AI Review):
  1. Check brand guidelines
  2. Check legal compliance
  3. Check platform policies
  4. Check accessibility
  5. Generate AI score
    ↓
Decision:
  - Score ≥ 85: Auto-approved
  - Score 70-84: Route to human review
  - Score < 70: Rejected
    ↓
If Human Review:
  Reviewer adds comments/annotations
    ↓
Approved
    ↓
Queue Job: release
    ↓
NestJS Worker:
  1. Prepare platform-specific formats
  2. Upload to YouTube
  3. Upload to TikTok
  4. Upload to Instagram
  5. Upload to LinkedIn
  6. Schedule/publish
    ↓
Monitor Analytics
    ↓
Track Incidents
```

---

## Deployment Architecture

### Local Development

```
docker-compose up -d
├── PostgreSQL (port 5432)
├── Redis (port 6379)
├── MinIO (port 9000)
├── Elasticsearch (port 9200)
├── FastAPI (port 8000)
├── NestJS (port 3001)
└── Next.js (port 3000)
```

### Production Deployment

**Frontend (Vercel):**
- Next.js application
- CDN distribution
- Automatic deployments from GitHub
- Environment variables via Vercel dashboard

**Backend (AWS/Docker):**
- FastAPI: ECS Fargate
- NestJS: ECS Fargate
- PostgreSQL: RDS
- Redis: ElastiCache
- S3: Object storage
- CloudFront: CDN

**Infrastructure as Code:**
```terraform
# Terraform configuration
resource "aws_ecs_cluster" "main" {
  name = "ghost-claw-os"
}

resource "aws_ecs_service" "fastapi" {
  name = "fastapi-service"
  cluster = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.fastapi.arn
  desired_count = 3
}

resource "aws_ecs_service" "nestjs" {
  name = "nestjs-service"
  cluster = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.nestjs.arn
  desired_count = 3
}

resource "aws_rds_cluster" "postgres" {
  cluster_identifier = "ghost-claw-os-db"
  engine = "aurora-postgresql"
  master_username = var.db_username
  master_password = var.db_password
}

resource "aws_elasticache_cluster" "redis" {
  cluster_id = "ghost-claw-os-redis"
  engine = "redis"
  node_type = "cache.t3.micro"
  num_cache_nodes = 3
}
```

---

## Security Architecture

### Authentication & Authorization

```
User Login
    ↓
OAuth2 (Google/GitHub)
    ↓
JWT Token Generation
    ↓
Store in Secure Cookie
    ↓
API Requests with JWT
    ↓
Verify JWT Signature
    ↓
Check User Permissions (RBAC)
    ↓
Grant/Deny Access
```

### Data Protection

**In Transit:**
- HTTPS/TLS 1.3 for all communications
- Certificate pinning for API calls

**At Rest:**
- AES-256 encryption for sensitive data
- Encrypted database connections
- Encrypted S3 objects

**PII Protection:**
- Tokenization for sensitive fields
- Encryption for email addresses
- Hashing for passwords

### Compliance & Auditing

- Audit logging for all user actions
- Compliance checks before publishing
- Regular security audits
- Penetration testing
- GDPR compliance (data deletion, export)

---

## Monitoring & Observability

### Logging

```
Application Logs → ELK Stack
├── Elasticsearch (storage)
├── Logstash (processing)
└── Kibana (visualization)
```

### Metrics

```
Application Metrics → Prometheus
├── Request latency
├── Error rates
├── Queue depth
├── Database connections
└── Memory/CPU usage
```

### Tracing

```
Distributed Tracing → Jaeger
├── Request flow across services
├── Latency breakdown
├── Error tracking
└── Performance analysis
```

### Alerting

```
Monitoring → AlertManager
├── Critical: Uptime < 99.9%
├── High: Error rate > 1%
├── Medium: Queue depth > 100
└── Low: Response time > 1s
```

---

## Scalability Strategy

### Horizontal Scaling

- **Frontend:** CDN + auto-scaling (Vercel)
- **Backend:** ECS auto-scaling based on CPU/memory
- **Database:** Read replicas + connection pooling
- **Queue:** Redis cluster for high throughput

### Vertical Scaling

- Increase instance size for CPU-intensive tasks
- Increase memory for caching
- Increase disk for storage

### Performance Optimization

- Database indexing on frequently queried columns
- Query optimization and caching
- Frontend code splitting and lazy loading
- Image optimization and compression
- CDN caching for static assets

---

**Document Version:** 1.0  
**Last Updated:** April 18, 2026  
**Status:** Ready for Implementation
