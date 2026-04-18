# Ghost Claw OS - API Routes Documentation

**Version:** 1.0  
**Status:** Design Phase  
**Author:** Manus AI  
**Date:** April 2026

---

## API Overview

Ghost Claw OS uses a hybrid backend architecture:
- **FastAPI (Python)**: Video processing, scene detection, transcription, asset management
- **NestJS (Node.js)**: Story generation, review gates, release management, monitoring

All APIs use REST with JSON payloads. Authentication is handled via JWT tokens.

---

## Authentication

### Login

```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}

Response:
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "User Name",
    "role": "creator"
  }
}
```

### OAuth (Google/GitHub)

```http
GET /api/auth/google/callback?code=auth_code&state=state_value

Response:
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "user": { ... }
}
```

### Refresh Token

```http
POST /api/auth/refresh
Authorization: Bearer {refresh_token}

Response:
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_in": 86400
}
```

---

## FastAPI Endpoints (Python Backend)

### Base URL: `http://localhost:8000/api`

### Video Processing

#### Upload Video

```http
POST /video/upload
Authorization: Bearer {token}
Content-Type: multipart/form-data

Form Data:
- file: <video_file>
- project_id: "uuid"
- topic: "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้"
- campaign: "Q1-2026-SME"
- target_audience: "SME owners"
- platforms: ["youtube", "tiktok", "instagram"]
- style_preference: "educational"
- brief: "Campaign brief text"

Response:
{
  "intake_id": "uuid",
  "project_id": "uuid",
  "video_url": "s3://bucket/uploads/video.mp4",
  "video_duration": 7200,
  "status": "uploaded",
  "created_at": "2026-04-18T10:00:00Z"
}
```

#### Start Video Processing

```http
POST /video/process
Authorization: Bearer {token}
Content-Type: application/json

{
  "intake_id": "uuid",
  "processing_type": "autocut",
  "options": {
    "scene_threshold": 0.4,
    "min_scene_duration": 5,
    "max_scenes": 20
  }
}

Response:
{
  "job_id": "uuid",
  "status": "queued",
  "queue_position": 3,
  "estimated_completion": "2026-04-18T11:00:00Z"
}
```

#### Get Job Status

```http
GET /video/jobs/{job_id}
Authorization: Bearer {token}

Response:
{
  "job_id": "uuid",
  "status": "processing",
  "progress": 0.45,
  "current_stage": "scene_detection",
  "started_at": "2026-04-18T10:15:00Z",
  "estimated_completion": "2026-04-18T10:45:00Z",
  "logs": [
    "2026-04-18T10:15:00Z: Started video processing",
    "2026-04-18T10:20:00Z: Generated proxy (480p)",
    "2026-04-18T10:25:00Z: Started scene detection"
  ]
}
```

#### Get Job Results

```http
GET /video/jobs/{job_id}/results
Authorization: Bearer {token}

Response:
{
  "job_id": "uuid",
  "status": "completed",
  "input": {
    "video_url": "s3://bucket/uploads/video.mp4",
    "duration": 7200
  },
  "output": {
    "stories": [
      {
        "story_id": "uuid",
        "start_time": 120,
        "end_time": 128,
        "duration": 8,
        "confidence": 0.92,
        "scene_type": "interview",
        "key_moments": ["moment1", "moment2"]
      },
      ...
    ],
    "metadata": {
      "total_scenes": 8,
      "avg_scene_duration": 8,
      "transcription": {
        "language": "th",
        "segments": [...]
      }
    }
  }
}
```

### Asset Management

#### List Assets

```http
GET /assets?tags=electricity&usage_role=hero&campaign=Q1-2026&limit=50&offset=0
Authorization: Bearer {token}

Response:
{
  "total": 245,
  "limit": 50,
  "offset": 0,
  "assets": [
    {
      "id": "uuid",
      "name": "SME-Electricity-Hero-1.jpg",
      "type": "image",
      "tags": ["electricity", "business", "sme"],
      "usage_role": "hero",
      "campaign_or_scene": "Q1-2026-SME",
      "drive_url": "https://drive.google.com/file/d/...",
      "preview_url": "https://cdn.example.com/preview.jpg",
      "usage_count": 5,
      "last_used": "2026-04-15T14:30:00Z",
      "created_at": "2026-01-10T08:00:00Z"
    },
    ...
  ]
}
```

#### Get Asset Details

```http
GET /assets/{asset_id}
Authorization: Bearer {token}

Response:
{
  "id": "uuid",
  "name": "SME-Electricity-Hero-1.jpg",
  "type": "image",
  "tags": ["electricity", "business", "sme"],
  "usage_role": "hero",
  "campaign_or_scene": "Q1-2026-SME",
  "drive_url": "https://drive.google.com/file/d/...",
  "preview_url": "https://cdn.example.com/preview.jpg",
  "metadata": {
    "width": 1920,
    "height": 1080,
    "size": 2048576,
    "format": "jpg",
    "created_date": "2026-01-10"
  },
  "usage_history": [
    {
      "project_id": "uuid",
      "story_id": "uuid",
      "used_at": "2026-04-15T14:30:00Z"
    }
  ]
}
```

#### Sync Google Drive Assets

```http
POST /assets/sync
Authorization: Bearer {token}
Content-Type: application/json

{
  "folder_id": "your_sirinx_folder_id",
  "sheet_id": "your_sirinx_media_asset_db_sheet_id"
}

Response:
{
  "sync_id": "uuid",
  "status": "in_progress",
  "total_assets": 1245,
  "synced_count": 0,
  "started_at": "2026-04-18T10:00:00Z"
}
```

#### Get Sync Status

```http
GET /assets/sync/{sync_id}
Authorization: Bearer {token}

Response:
{
  "sync_id": "uuid",
  "status": "completed",
  "total_assets": 1245,
  "synced_count": 1245,
  "new_assets": 45,
  "updated_assets": 120,
  "deleted_assets": 5,
  "started_at": "2026-04-18T10:00:00Z",
  "completed_at": "2026-04-18T10:15:00Z"
}
```

### Health & Status

#### Health Check

```http
GET /health

Response:
{
  "status": "healthy",
  "timestamp": "2026-04-18T10:00:00Z",
  "services": {
    "database": "connected",
    "redis": "connected",
    "s3": "connected",
    "google_drive": "connected"
  }
}
```

---

## NestJS Endpoints (Node.js Backend)

### Base URL: `http://localhost:3001/api`

### Authentication

#### Login

```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}

Response: (same as FastAPI)
```

### Projects

#### Create Project

```http
POST /projects
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Q1 2026 SME Campaign",
  "description": "Educational content about SME electricity costs",
  "campaign": "Q1-2026-SME",
  "team_members": ["user_id_1", "user_id_2"]
}

Response:
{
  "id": "uuid",
  "name": "Q1 2026 SME Campaign",
  "description": "Educational content about SME electricity costs",
  "campaign": "Q1-2026-SME",
  "owner_id": "uuid",
  "team_members": ["user_id_1", "user_id_2"],
  "status": "draft",
  "created_at": "2026-04-18T10:00:00Z",
  "updated_at": "2026-04-18T10:00:00Z"
}
```

#### List Projects

```http
GET /projects?status=in-progress&limit=20&offset=0
Authorization: Bearer {token}

Response:
{
  "total": 45,
  "limit": 20,
  "offset": 0,
  "projects": [
    {
      "id": "uuid",
      "name": "Q1 2026 SME Campaign",
      "description": "...",
      "campaign": "Q1-2026-SME",
      "status": "in-progress",
      "created_at": "2026-04-18T10:00:00Z",
      "updated_at": "2026-04-18T10:00:00Z"
    },
    ...
  ]
}
```

#### Get Project Details

```http
GET /projects/{project_id}
Authorization: Bearer {token}

Response:
{
  "id": "uuid",
  "name": "Q1 2026 SME Campaign",
  "description": "...",
  "campaign": "Q1-2026-SME",
  "owner_id": "uuid",
  "team_members": [...],
  "status": "in-progress",
  "stories": [
    {
      "id": "uuid",
      "title": "Story 1",
      "status": "approved",
      "created_at": "2026-04-18T10:00:00Z"
    }
  ],
  "created_at": "2026-04-18T10:00:00Z",
  "updated_at": "2026-04-18T10:00:00Z"
}
```

#### Update Project

```http
PATCH /projects/{project_id}
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Q1 2026 SME Campaign - Updated",
  "status": "completed"
}

Response:
{
  "id": "uuid",
  "name": "Q1 2026 SME Campaign - Updated",
  "status": "completed",
  "updated_at": "2026-04-18T11:00:00Z"
}
```

### Stories

#### Generate Story (One-Click Story Engine)

```http
POST /stories/generate
Authorization: Bearer {token}
Content-Type: application/json

{
  "project_id": "uuid",
  "topic": "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้",
  "platforms": ["youtube", "tiktok", "instagram"],
  "style": "educational",
  "target_audience": "SME owners",
  "cta": "Subscribe for more insights"
}

Response:
{
  "generation_id": "uuid",
  "project_id": "uuid",
  "status": "queued",
  "topic": "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้",
  "estimated_completion": "2026-04-18T10:05:00Z",
  "created_at": "2026-04-18T10:00:00Z"
}
```

#### Get Generation Status

```http
GET /stories/generate/{generation_id}
Authorization: Bearer {token}

Response:
{
  "generation_id": "uuid",
  "status": "processing",
  "progress": 0.65,
  "current_stage": "visual_generation",
  "stories_generated": 5,
  "total_stories": 8,
  "estimated_completion": "2026-04-18T10:05:00Z",
  "logs": [
    "2026-04-18T10:00:00Z: Started story generation",
    "2026-04-18T10:01:00Z: Generated hook",
    "2026-04-18T10:02:00Z: Generated core angle",
    "2026-04-18T10:03:00Z: Generating visual prompts"
  ]
}
```

#### Get Generated Stories

```http
GET /stories/generate/{generation_id}/results
Authorization: Bearer {token}

Response:
{
  "generation_id": "uuid",
  "status": "completed",
  "stories": [
    {
      "story_id": "uuid",
      "title": "Why SME Electricity Costs Are Unpredictable",
      "description": "Learn about the factors affecting SME electricity bills",
      "duration": 8,
      "video_url": "s3://bucket/stories/story-1.mp4",
      "thumbnail_url": "s3://bucket/thumbnails/story-1.jpg",
      "captions": [
        {
          "language": "th",
          "url": "s3://bucket/captions/story-1-th.vtt"
        },
        {
          "language": "en",
          "url": "s3://bucket/captions/story-1-en.vtt"
        }
      ],
      "cta": "Subscribe for more insights",
      "metadata": {
        "hook": "Did you know SME electricity costs can vary by 40%?",
        "core_angle": "Understanding unpredictable electricity pricing",
        "scenes": [...]
      }
    },
    ...
  ]
}
```

#### List Stories

```http
GET /stories?project_id=uuid&status=approved&limit=20
Authorization: Bearer {token}

Response:
{
  "total": 45,
  "limit": 20,
  "stories": [
    {
      "id": "uuid",
      "project_id": "uuid",
      "title": "Story Title",
      "duration": 8,
      "status": "approved",
      "created_at": "2026-04-18T10:00:00Z"
    },
    ...
  ]
}
```

#### Get Story Details

```http
GET /stories/{story_id}
Authorization: Bearer {token}

Response:
{
  "id": "uuid",
  "project_id": "uuid",
  "title": "Story Title",
  "description": "Story description",
  "duration": 8,
  "video_url": "s3://bucket/stories/story-1.mp4",
  "thumbnail_url": "s3://bucket/thumbnails/story-1.jpg",
  "captions": [...],
  "cta": "Subscribe",
  "status": "approved",
  "created_at": "2026-04-18T10:00:00Z",
  "updated_at": "2026-04-18T10:00:00Z"
}
```

### Prompts (Prompt Lab)

#### Create Prompt

```http
POST /prompts
Authorization: Bearer {token}
Content-Type: application/json

{
  "project_id": "uuid",
  "name": "SME Electricity - Educational",
  "dimensions": {
    "genre": "educational",
    "tone": "professional",
    "pacing": "moderate",
    "visual_style": "cinematic",
    "target_audience": "SME owners",
    "platform": "youtube",
    "brand_voice": "authoritative",
    "emotional_arc": "curiosity → tension → resolution → action",
    "cta": "Subscribe for more insights"
  }
}

Response:
{
  "id": "uuid",
  "project_id": "uuid",
  "name": "SME Electricity - Educational",
  "dimensions": {...},
  "version": 1,
  "created_at": "2026-04-18T10:00:00Z"
}
```

#### Generate Prompt Variations

```http
POST /prompts/{prompt_id}/generate-variations
Authorization: Bearer {token}
Content-Type: application/json

{
  "count": 5,
  "adapter": "flow"  // or "grok"
}

Response:
{
  "prompt_id": "uuid",
  "variations": [
    {
      "variation_id": "uuid",
      "adapter": "flow",
      "generated_prompt": "Detailed prompt text...",
      "created_at": "2026-04-18T10:00:00Z"
    },
    ...
  ]
}
```

### Reviews (Review Gate)

#### Submit for Review

```http
POST /reviews/submit
Authorization: Bearer {token}
Content-Type: application/json

{
  "story_id": "uuid",
  "submission_type": "ai_review"  // or "human_review"
}

Response:
{
  "review_id": "uuid",
  "story_id": "uuid",
  "status": "in_progress",
  "submission_type": "ai_review",
  "created_at": "2026-04-18T10:00:00Z"
}
```

#### Get Review Status

```http
GET /reviews/{review_id}
Authorization: Bearer {token}

Response:
{
  "review_id": "uuid",
  "story_id": "uuid",
  "status": "completed",
  "submission_type": "ai_review",
  "ai_score": 0.87,
  "issues": [
    {
      "type": "brand_guideline",
      "severity": "warning",
      "message": "Logo placement could be improved",
      "timestamp": 3.5
    }
  ],
  "recommendation": "approved_with_notes",
  "completed_at": "2026-04-18T10:02:00Z"
}
```

#### Approve/Reject Story

```http
POST /reviews/{review_id}/approve
Authorization: Bearer {token}
Content-Type: application/json

{
  "comments": "Looks great, ready for release",
  "approved_by": "reviewer_id"
}

Response:
{
  "review_id": "uuid",
  "story_id": "uuid",
  "status": "approved",
  "approved_at": "2026-04-18T10:05:00Z"
}
```

### Releases (Release Center)

#### Create Release

```http
POST /releases
Authorization: Bearer {token}
Content-Type: application/json

{
  "project_id": "uuid",
  "story_ids": ["uuid1", "uuid2", "uuid3"],
  "platforms": ["youtube", "tiktok", "instagram"],
  "scheduled_for": "2026-04-20T10:00:00Z",
  "release_packet_name": "Q1-SME-Campaign-Release-1"
}

Response:
{
  "release_id": "uuid",
  "project_id": "uuid",
  "story_ids": ["uuid1", "uuid2", "uuid3"],
  "platforms": ["youtube", "tiktok", "instagram"],
  "status": "scheduled",
  "scheduled_for": "2026-04-20T10:00:00Z",
  "created_at": "2026-04-18T10:00:00Z"
}
```

#### Get Release Status

```http
GET /releases/{release_id}
Authorization: Bearer {token}

Response:
{
  "release_id": "uuid",
  "project_id": "uuid",
  "story_ids": ["uuid1", "uuid2", "uuid3"],
  "platforms": ["youtube", "tiktok", "instagram"],
  "status": "released",
  "scheduled_for": "2026-04-20T10:00:00Z",
  "released_at": "2026-04-20T10:00:15Z",
  "platform_status": {
    "youtube": {
      "status": "published",
      "url": "https://youtube.com/watch?v=...",
      "published_at": "2026-04-20T10:00:15Z"
    },
    "tiktok": {
      "status": "published",
      "url": "https://tiktok.com/@user/video/...",
      "published_at": "2026-04-20T10:00:20Z"
    },
    "instagram": {
      "status": "published",
      "url": "https://instagram.com/p/...",
      "published_at": "2026-04-20T10:00:25Z"
    }
  }
}
```

### Monitoring (Watch Center)

#### System Health

```http
GET /system/health
Authorization: Bearer {token}

Response:
{
  "status": "healthy",
  "timestamp": "2026-04-18T10:00:00Z",
  "components": {
    "database": {
      "status": "healthy",
      "response_time_ms": 5
    },
    "redis": {
      "status": "healthy",
      "response_time_ms": 2
    },
    "s3": {
      "status": "healthy",
      "response_time_ms": 150
    },
    "google_drive": {
      "status": "healthy",
      "response_time_ms": 300
    }
  }
}
```

#### Queue Status

```http
GET /system/queue/status
Authorization: Bearer {token}

Response:
{
  "timestamp": "2026-04-18T10:00:00Z",
  "queues": {
    "video-processing": {
      "total_jobs": 15,
      "active_jobs": 4,
      "queued_jobs": 11,
      "failed_jobs": 0,
      "avg_processing_time_seconds": 1200
    },
    "story-generation": {
      "total_jobs": 8,
      "active_jobs": 2,
      "queued_jobs": 6,
      "failed_jobs": 0,
      "avg_processing_time_seconds": 300
    },
    "rendering": {
      "total_jobs": 20,
      "active_jobs": 8,
      "queued_jobs": 12,
      "failed_jobs": 0,
      "avg_processing_time_seconds": 600
    },
    "review": {
      "total_jobs": 5,
      "active_jobs": 1,
      "queued_jobs": 4,
      "failed_jobs": 0,
      "avg_processing_time_seconds": 120
    },
    "release": {
      "total_jobs": 3,
      "active_jobs": 1,
      "queued_jobs": 2,
      "failed_jobs": 0,
      "avg_processing_time_seconds": 300
    }
  }
}
```

#### Incidents

```http
GET /system/incidents?limit=20
Authorization: Bearer {token}

Response:
{
  "total": 5,
  "incidents": [
    {
      "incident_id": "uuid",
      "type": "job_failure",
      "severity": "high",
      "message": "Video processing job failed: FFmpeg error",
      "affected_resource": "job_id_123",
      "created_at": "2026-04-18T09:45:00Z",
      "resolved_at": "2026-04-18T09:50:00Z",
      "resolution": "Job retried successfully"
    },
    ...
  ]
}
```

#### Performance Metrics

```http
GET /system/metrics?period=24h
Authorization: Bearer {token}

Response:
{
  "period": "24h",
  "timestamp": "2026-04-18T10:00:00Z",
  "metrics": {
    "api_requests": {
      "total": 45000,
      "success_rate": 0.98,
      "avg_response_time_ms": 145,
      "p95_response_time_ms": 320,
      "p99_response_time_ms": 850
    },
    "video_processing": {
      "total_videos": 120,
      "success_rate": 0.95,
      "avg_processing_time_seconds": 1200,
      "total_hours_processed": 240
    },
    "story_generation": {
      "total_stories": 960,
      "success_rate": 0.97,
      "avg_generation_time_seconds": 300,
      "avg_quality_score": 8.2
    },
    "rendering": {
      "total_renders": 1920,
      "success_rate": 0.96,
      "avg_render_time_seconds": 600
    }
  }
}
```

---

## WebSocket Events

### Connection

```javascript
// Client
const socket = io('http://localhost:3001', {
  auth: {
    token: access_token
  }
});

socket.on('connect', () => {
  console.log('Connected to server');
});
```

### Job Progress Updates

```javascript
// Server emits
socket.emit('job:progress', {
  job_id: 'uuid',
  status: 'processing',
  progress: 0.45,
  current_stage: 'scene_detection'
});

// Client listens
socket.on('job:progress', (data) => {
  console.log(`Job ${data.job_id} is ${data.progress * 100}% complete`);
});
```

### Notifications

```javascript
// Server emits
socket.emit('notification', {
  type: 'job_completed',
  title: 'Story generation completed',
  message: 'Your story pack is ready for review',
  data: {
    project_id: 'uuid',
    generation_id: 'uuid'
  }
});

// Client listens
socket.on('notification', (data) => {
  console.log(data.message);
});
```

---

## Error Responses

### Standard Error Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input parameters",
    "details": [
      {
        "field": "topic",
        "message": "Topic is required"
      }
    ]
  }
}
```

### Common Error Codes

| Code | Status | Description |
|------|--------|-------------|
| `UNAUTHORIZED` | 401 | Missing or invalid authentication token |
| `FORBIDDEN` | 403 | User doesn't have permission for this resource |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 400 | Invalid input parameters |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server error |
| `SERVICE_UNAVAILABLE` | 503 | Service temporarily unavailable |

---

**Document Version:** 1.0  
**Last Updated:** April 18, 2026  
**Status:** Ready for Implementation
