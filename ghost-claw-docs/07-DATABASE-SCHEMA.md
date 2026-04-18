# Ghost Claw OS - Database Schema

**Version:** 1.0  
**Status:** Design Phase  
**Database:** PostgreSQL 16  
**Author:** Manus AI  
**Date:** April 2026

---

## Schema Overview

```sql
-- Users & Authentication
users
user_sessions
user_roles
permissions

-- Projects & Organization
projects
project_members
project_templates

-- Content Intake
intakes
intake_metadata
brief_documents

-- Video Processing
video_jobs
video_metadata
scenes
transcriptions
key_moments

-- Stories & Generation
stories
story_scenes
story_captions
story_metadata
generation_jobs
generation_logs

-- Reviews & Approvals
reviews
review_issues
review_annotations
approval_workflows

-- Releases & Publishing
releases
release_stories
platform_publish_status
release_analytics

-- Assets
assets
asset_tags
asset_usage
asset_metadata

-- Prompts
prompts
prompt_dimensions
prompt_variations

-- Queue & Jobs
jobs
job_logs
job_failures

-- Monitoring
system_incidents
system_metrics
audit_logs
```

---

## Detailed Schema

### Users & Authentication

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  password_hash VARCHAR(255),
  avatar_url VARCHAR(500),
  role ENUM('admin', 'creator', 'reviewer', 'viewer') DEFAULT 'creator',
  subscription_tier ENUM('free', 'pro', 'enterprise') DEFAULT 'free',
  status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login TIMESTAMP,
  
  CONSTRAINT valid_email CHECK (email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$')
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_status ON users(status);

-- User sessions
CREATE TABLE user_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  access_token VARCHAR(500) NOT NULL,
  refresh_token VARCHAR(500) NOT NULL,
  ip_address INET,
  user_agent VARCHAR(500),
  expires_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_tokens CHECK (access_token != '' AND refresh_token != '')
);

CREATE INDEX idx_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_sessions_expires_at ON user_sessions(expires_at);

-- User roles (for fine-grained permissions)
CREATE TABLE user_roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role_name VARCHAR(100) NOT NULL,
  project_id UUID,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id, role_name, project_id)
);

-- Permissions
CREATE TABLE permissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  role_name VARCHAR(100) NOT NULL,
  permission VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(role_name, permission)
);
```

### Projects & Organization

```sql
-- Projects
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  campaign VARCHAR(255),
  owner_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  template_id UUID,
  status ENUM('draft', 'in-progress', 'completed', 'archived') DEFAULT 'draft',
  visibility ENUM('private', 'team', 'public') DEFAULT 'private',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_name CHECK (name != '')
);

CREATE INDEX idx_projects_owner_id ON projects(owner_id);
CREATE INDEX idx_projects_campaign ON projects(campaign);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_created_at ON projects(created_at);

-- Project members
CREATE TABLE project_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role ENUM('owner', 'editor', 'reviewer', 'viewer') DEFAULT 'viewer',
  added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(project_id, user_id)
);

CREATE INDEX idx_project_members_project_id ON project_members(project_id);
CREATE INDEX idx_project_members_user_id ON project_members(user_id);

-- Project templates
CREATE TABLE project_templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  config JSONB NOT NULL,
  created_by UUID NOT NULL REFERENCES users(id),
  is_public BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_config CHECK (config != 'null'::jsonb)
);
```

### Content Intake

```sql
-- Intakes
CREATE TABLE intakes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  video_url VARCHAR(500) NOT NULL,
  video_duration INTEGER,  -- in seconds
  video_size BIGINT,       -- in bytes
  topic VARCHAR(500) NOT NULL,
  campaign VARCHAR(255),
  target_audience VARCHAR(255),
  platforms VARCHAR[] DEFAULT ARRAY[]::VARCHAR[],
  style_preference VARCHAR(100),
  brief TEXT,
  status ENUM('uploaded', 'processing', 'completed', 'failed') DEFAULT 'uploaded',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_url CHECK (video_url ~ '^(https?://|s3://)')
);

CREATE INDEX idx_intakes_project_id ON intakes(project_id);
CREATE INDEX idx_intakes_status ON intakes(status);
CREATE INDEX idx_intakes_created_at ON intakes(created_at);

-- Intake metadata
CREATE TABLE intake_metadata (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  intake_id UUID NOT NULL REFERENCES intakes(id) ON DELETE CASCADE,
  key VARCHAR(100) NOT NULL,
  value TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(intake_id, key)
);

-- Brief documents
CREATE TABLE brief_documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  intake_id UUID NOT NULL REFERENCES intakes(id) ON DELETE CASCADE,
  title VARCHAR(255),
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Video Processing

```sql
-- Video jobs
CREATE TABLE video_jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  intake_id UUID NOT NULL REFERENCES intakes(id) ON DELETE CASCADE,
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  job_type ENUM('proxy_generation', 'scene_detection', 'transcription', 'key_moment_extraction') NOT NULL,
  status ENUM('queued', 'processing', 'completed', 'failed') DEFAULT 'queued',
  progress FLOAT DEFAULT 0,
  current_stage VARCHAR(100),
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  error_message TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_progress CHECK (progress >= 0 AND progress <= 1)
);

CREATE INDEX idx_video_jobs_intake_id ON video_jobs(intake_id);
CREATE INDEX idx_video_jobs_project_id ON video_jobs(project_id);
CREATE INDEX idx_video_jobs_status ON video_jobs(status);
CREATE INDEX idx_video_jobs_created_at ON video_jobs(created_at);

-- Video metadata
CREATE TABLE video_metadata (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  intake_id UUID NOT NULL REFERENCES intakes(id) ON DELETE CASCADE,
  proxy_url VARCHAR(500),
  proxy_duration INTEGER,
  resolution VARCHAR(20),
  fps FLOAT,
  codec VARCHAR(50),
  bitrate INTEGER,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(intake_id)
);

-- Scenes
CREATE TABLE scenes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  intake_id UUID NOT NULL REFERENCES intakes(id) ON DELETE CASCADE,
  video_job_id UUID NOT NULL REFERENCES video_jobs(id) ON DELETE CASCADE,
  start_time FLOAT NOT NULL,  -- in seconds
  end_time FLOAT NOT NULL,
  duration FLOAT NOT NULL,
  confidence FLOAT,
  scene_type VARCHAR(50),
  description TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_times CHECK (start_time < end_time)
);

CREATE INDEX idx_scenes_intake_id ON scenes(intake_id);
CREATE INDEX idx_scenes_video_job_id ON scenes(video_job_id);
CREATE INDEX idx_scenes_start_time ON scenes(start_time);

-- Transcriptions
CREATE TABLE transcriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  intake_id UUID NOT NULL REFERENCES intakes(id) ON DELETE CASCADE,
  video_job_id UUID NOT NULL REFERENCES video_jobs(id) ON DELETE CASCADE,
  language VARCHAR(10) NOT NULL,
  full_text TEXT NOT NULL,
  segments JSONB NOT NULL,  -- Array of {start, end, text, speaker}
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(intake_id, language)
);

CREATE INDEX idx_transcriptions_intake_id ON transcriptions(intake_id);
CREATE INDEX idx_transcriptions_language ON transcriptions(language);

-- Key moments
CREATE TABLE key_moments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  intake_id UUID NOT NULL REFERENCES intakes(id) ON DELETE CASCADE,
  video_job_id UUID NOT NULL REFERENCES video_jobs(id) ON DELETE CASCADE,
  start_time FLOAT NOT NULL,
  end_time FLOAT NOT NULL,
  importance_score FLOAT NOT NULL,
  reason VARCHAR(255),
  tags VARCHAR[] DEFAULT ARRAY[]::VARCHAR[],
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_score CHECK (importance_score >= 0 AND importance_score <= 1)
);

CREATE INDEX idx_key_moments_intake_id ON key_moments(intake_id);
CREATE INDEX idx_key_moments_importance_score ON key_moments(importance_score DESC);
```

### Stories & Generation

```sql
-- Stories
CREATE TABLE stories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  intake_id UUID,  -- NULL if generated from One-Click Story Engine
  generation_job_id UUID,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  video_url VARCHAR(500),
  thumbnail_url VARCHAR(500),
  duration INTEGER,  -- in seconds
  cta VARCHAR(255),
  status ENUM('draft', 'in-review', 'approved', 'released', 'archived') DEFAULT 'draft',
  quality_score FLOAT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_duration CHECK (duration > 0)
);

CREATE INDEX idx_stories_project_id ON stories(project_id);
CREATE INDEX idx_stories_status ON stories(status);
CREATE INDEX idx_stories_created_at ON stories(created_at);

-- Story scenes (breakdown of story into scenes)
CREATE TABLE story_scenes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  story_id UUID NOT NULL REFERENCES stories(id) ON DELETE CASCADE,
  scene_number INTEGER NOT NULL,
  title VARCHAR(255),
  description TEXT,
  start_time FLOAT,
  end_time FLOAT,
  visual_prompt TEXT,
  voiceover_script TEXT,
  camera_direction VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(story_id, scene_number)
);

-- Story captions
CREATE TABLE story_captions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  story_id UUID NOT NULL REFERENCES stories(id) ON DELETE CASCADE,
  language VARCHAR(10) NOT NULL,
  caption_url VARCHAR(500),
  caption_text TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(story_id, language)
);

-- Story metadata
CREATE TABLE story_metadata (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  story_id UUID NOT NULL REFERENCES stories(id) ON DELETE CASCADE,
  hook VARCHAR(500),
  core_angle VARCHAR(500),
  target_audience VARCHAR(255),
  platforms VARCHAR[] DEFAULT ARRAY[]::VARCHAR[],
  tags VARCHAR[] DEFAULT ARRAY[]::VARCHAR[],
  metadata JSONB,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(story_id)
);

-- Generation jobs (for One-Click Story Engine)
CREATE TABLE generation_jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  topic VARCHAR(500) NOT NULL,
  platforms VARCHAR[] NOT NULL,
  style VARCHAR(100),
  target_audience VARCHAR(255),
  cta VARCHAR(255),
  status ENUM('queued', 'processing', 'completed', 'failed') DEFAULT 'queued',
  progress FLOAT DEFAULT 0,
  current_stage VARCHAR(100),
  stories_generated INTEGER DEFAULT 0,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  error_message TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_progress CHECK (progress >= 0 AND progress <= 1)
);

CREATE INDEX idx_generation_jobs_project_id ON generation_jobs(project_id);
CREATE INDEX idx_generation_jobs_status ON generation_jobs(status);
CREATE INDEX idx_generation_jobs_created_at ON generation_jobs(created_at);

-- Generation logs
CREATE TABLE generation_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  generation_job_id UUID NOT NULL REFERENCES generation_jobs(id) ON DELETE CASCADE,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  stage VARCHAR(100),
  message TEXT,
  level ENUM('info', 'warning', 'error') DEFAULT 'info'
);

CREATE INDEX idx_generation_logs_generation_job_id ON generation_logs(generation_job_id);
```

### Reviews & Approvals

```sql
-- Reviews
CREATE TABLE reviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  story_id UUID NOT NULL REFERENCES stories(id) ON DELETE CASCADE,
  reviewer_id UUID REFERENCES users(id) ON DELETE SET NULL,
  review_type ENUM('ai_review', 'human_review') NOT NULL,
  status ENUM('in_progress', 'approved', 'rejected', 'needs_revision') DEFAULT 'in_progress',
  ai_score FLOAT,
  comments TEXT,
  submitted_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_score CHECK (ai_score IS NULL OR (ai_score >= 0 AND ai_score <= 1))
);

CREATE INDEX idx_reviews_story_id ON reviews(story_id);
CREATE INDEX idx_reviews_reviewer_id ON reviews(reviewer_id);
CREATE INDEX idx_reviews_status ON reviews(status);
CREATE INDEX idx_reviews_created_at ON reviews(created_at);

-- Review issues
CREATE TABLE review_issues (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  review_id UUID NOT NULL REFERENCES reviews(id) ON DELETE CASCADE,
  issue_type VARCHAR(100) NOT NULL,  -- e.g., 'brand_guideline', 'legal', 'platform_policy'
  severity ENUM('info', 'warning', 'error') DEFAULT 'warning',
  message TEXT NOT NULL,
  timestamp FLOAT,  -- in seconds (for video issues)
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_review_issues_review_id ON review_issues(review_id);
CREATE INDEX idx_review_issues_issue_type ON review_issues(issue_type);

-- Review annotations
CREATE TABLE review_annotations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  review_id UUID NOT NULL REFERENCES reviews(id) ON DELETE CASCADE,
  annotation_type VARCHAR(100),
  start_time FLOAT,
  end_time FLOAT,
  comment TEXT,
  created_by UUID NOT NULL REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Approval workflows
CREATE TABLE approval_workflows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  story_id UUID NOT NULL REFERENCES stories(id) ON DELETE CASCADE,
  step_number INTEGER NOT NULL,
  reviewer_id UUID REFERENCES users(id),
  status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
  comments TEXT,
  completed_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(story_id, step_number)
);
```

### Releases & Publishing

```sql
-- Releases
CREATE TABLE releases (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  release_name VARCHAR(255) NOT NULL,
  status ENUM('draft', 'scheduled', 'released', 'failed') DEFAULT 'draft',
  scheduled_for TIMESTAMP,
  released_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_releases_project_id ON releases(project_id);
CREATE INDEX idx_releases_status ON releases(status);
CREATE INDEX idx_releases_scheduled_for ON releases(scheduled_for);

-- Release stories
CREATE TABLE release_stories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  release_id UUID NOT NULL REFERENCES releases(id) ON DELETE CASCADE,
  story_id UUID NOT NULL REFERENCES stories(id) ON DELETE CASCADE,
  platform VARCHAR(50) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(release_id, story_id, platform)
);

-- Platform publish status
CREATE TABLE platform_publish_status (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  release_id UUID NOT NULL REFERENCES releases(id) ON DELETE CASCADE,
  platform VARCHAR(50) NOT NULL,
  status ENUM('pending', 'publishing', 'published', 'failed') DEFAULT 'pending',
  platform_url VARCHAR(500),
  platform_id VARCHAR(255),
  error_message TEXT,
  published_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(release_id, platform)
);

-- Release analytics
CREATE TABLE release_analytics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  release_id UUID NOT NULL REFERENCES releases(id) ON DELETE CASCADE,
  platform VARCHAR(50) NOT NULL,
  views INTEGER DEFAULT 0,
  likes INTEGER DEFAULT 0,
  comments INTEGER DEFAULT 0,
  shares INTEGER DEFAULT 0,
  click_through_rate FLOAT,
  engagement_rate FLOAT,
  collected_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_release_analytics_release_id ON release_analytics(release_id);
CREATE INDEX idx_release_analytics_platform ON release_analytics(platform);
```

### Assets

```sql
-- Assets
CREATE TABLE assets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  type ENUM('image', 'video', 'audio', 'template') NOT NULL,
  drive_url VARCHAR(500) NOT NULL,
  preview_url VARCHAR(500),
  usage_role VARCHAR(100),  -- e.g., 'hero', 'supporting', 'transition'
  campaign_or_scene VARCHAR(255),
  usage_count INTEGER DEFAULT 0,
  last_used TIMESTAMP,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_assets_type ON assets(type);
CREATE INDEX idx_assets_usage_role ON assets(usage_role);
CREATE INDEX idx_assets_campaign_or_scene ON assets(campaign_or_scene);
CREATE INDEX idx_assets_created_at ON assets(created_at);

-- Asset tags
CREATE TABLE asset_tags (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  asset_id UUID NOT NULL REFERENCES assets(id) ON DELETE CASCADE,
  tag VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(asset_id, tag)
);

CREATE INDEX idx_asset_tags_asset_id ON asset_tags(asset_id);
CREATE INDEX idx_asset_tags_tag ON asset_tags(tag);

-- Asset usage
CREATE TABLE asset_usage (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  asset_id UUID NOT NULL REFERENCES assets(id) ON DELETE CASCADE,
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  story_id UUID REFERENCES stories(id) ON DELETE SET NULL,
  used_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_asset_usage_asset_id ON asset_usage(asset_id);
CREATE INDEX idx_asset_usage_project_id ON asset_usage(project_id);
CREATE INDEX idx_asset_usage_story_id ON asset_usage(story_id);

-- Asset metadata
CREATE TABLE asset_metadata (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  asset_id UUID NOT NULL REFERENCES assets(id) ON DELETE CASCADE,
  key VARCHAR(100) NOT NULL,
  value TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(asset_id, key)
);
```

### Prompts

```sql
-- Prompts
CREATE TABLE prompts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  version INTEGER DEFAULT 1,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_prompts_project_id ON prompts(project_id);

-- Prompt dimensions
CREATE TABLE prompt_dimensions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  prompt_id UUID NOT NULL REFERENCES prompts(id) ON DELETE CASCADE,
  dimension_name VARCHAR(100) NOT NULL,
  dimension_value VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(prompt_id, dimension_name)
);

-- Prompt variations
CREATE TABLE prompt_variations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  prompt_id UUID NOT NULL REFERENCES prompts(id) ON DELETE CASCADE,
  adapter VARCHAR(50) NOT NULL,  -- 'flow' or 'grok'
  generated_prompt TEXT NOT NULL,
  quality_score FLOAT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_prompt_variations_prompt_id ON prompt_variations(prompt_id);
CREATE INDEX idx_prompt_variations_adapter ON prompt_variations(adapter);
```

### Queue & Jobs

```sql
-- Jobs (generic job tracking)
CREATE TABLE jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  queue_name VARCHAR(100) NOT NULL,
  job_type VARCHAR(100) NOT NULL,
  status ENUM('queued', 'processing', 'completed', 'failed', 'retrying') DEFAULT 'queued',
  payload JSONB NOT NULL,
  result JSONB,
  error_message TEXT,
  retry_count INTEGER DEFAULT 0,
  max_retries INTEGER DEFAULT 3,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT valid_retries CHECK (retry_count <= max_retries)
);

CREATE INDEX idx_jobs_queue_name ON jobs(queue_name);
CREATE INDEX idx_jobs_status ON jobs(status);
CREATE INDEX idx_jobs_created_at ON jobs(created_at);

-- Job logs
CREATE TABLE job_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  job_id UUID NOT NULL REFERENCES jobs(id) ON DELETE CASCADE,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  message TEXT,
  level ENUM('info', 'warning', 'error') DEFAULT 'info'
);

CREATE INDEX idx_job_logs_job_id ON job_logs(job_id);

-- Job failures
CREATE TABLE job_failures (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  job_id UUID NOT NULL REFERENCES jobs(id) ON DELETE CASCADE,
  error_message TEXT NOT NULL,
  stack_trace TEXT,
  failed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_job_failures_job_id ON job_failures(job_id);
```

### Monitoring

```sql
-- System incidents
CREATE TABLE system_incidents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  incident_type VARCHAR(100) NOT NULL,
  severity ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
  title VARCHAR(255) NOT NULL,
  description TEXT,
  affected_resource VARCHAR(255),
  status ENUM('open', 'investigating', 'resolved') DEFAULT 'open',
  resolved_at TIMESTAMP,
  resolution_notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_incidents_severity ON system_incidents(severity);
CREATE INDEX idx_incidents_status ON system_incidents(status);
CREATE INDEX idx_incidents_created_at ON system_incidents(created_at);

-- System metrics
CREATE TABLE system_metrics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  metric_name VARCHAR(100) NOT NULL,
  metric_value FLOAT NOT NULL,
  tags JSONB,
  collected_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_metrics_metric_name ON system_metrics(metric_name);
CREATE INDEX idx_metrics_collected_at ON system_metrics(collected_at);

-- Audit logs
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE SET NULL,
  action VARCHAR(100) NOT NULL,
  resource_type VARCHAR(100),
  resource_id UUID,
  changes JSONB,
  ip_address INET,
  user_agent VARCHAR(500),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at);
```

---

## Indexes & Performance

### Key Indexes

```sql
-- Composite indexes for common queries
CREATE INDEX idx_projects_owner_status ON projects(owner_id, status);
CREATE INDEX idx_stories_project_status ON stories(project_id, status);
CREATE INDEX idx_reviews_story_status ON reviews(story_id, status);
CREATE INDEX idx_jobs_queue_status ON jobs(queue_name, status);
CREATE INDEX idx_asset_tags_tag_asset ON asset_tags(tag, asset_id);
```

### Query Optimization

- **Partitioning**: Consider partitioning large tables by date (e.g., `video_jobs`, `stories`)
- **Materialized Views**: Create for frequently accessed aggregations
- **Connection Pooling**: Use PgBouncer with 100-200 connections
- **Vacuum & Analyze**: Run regularly to maintain index efficiency

---

## Migrations

### Initial Migration

```bash
# Run all migrations
pnpm db:migrate

# Create specific migration
pnpm db:migrate:create add_new_table

# Rollback last migration
pnpm db:migrate:rollback
```

---

**Document Version:** 1.0  
**Last Updated:** April 18, 2026  
**Status:** Ready for Implementation
