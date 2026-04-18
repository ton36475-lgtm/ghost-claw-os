# Ghost Claw OS - Queue & Worker Design

**Version:** 1.0  
**Status:** Design Phase  
**Author:** Manus AI  
**Date:** April 2026

---

## Overview

Ghost Claw OS uses **BullMQ** (Node.js job queue) + **Redis** for distributed job processing. This architecture enables:

- **Horizontal Scaling**: Add workers to handle more jobs
- **Fault Tolerance**: Automatic retry on failure
- **Progress Tracking**: Real-time job progress updates
- **Priority Queues**: High-priority jobs processed first
- **Dead Letter Queues**: Failed jobs for investigation

---

## Queue Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    API Requests                             │
│  (FastAPI, NestJS, Next.js)                                │
└────────────────┬────────────────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────────────────┐
│                  Redis Queue Backend                        │
│  ┌──────────────┬──────────────┬──────────────┐             │
│  │ video-proc   │ story-gen    │ rendering    │             │
│  │ review       │ release      │ (5 queues)   │             │
│  └──────────────┴──────────────┴──────────────┘             │
└────────────────┬────────────────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────────────────┐
│                   Worker Pool                               │
│  ┌──────────────┬──────────────┬──────────────┐             │
│  │ Video Worker │ Story Worker │ Render Worker│             │
│  │ (4-8)        │ (2-4)        │ (4-8)        │             │
│  │ (FastAPI)    │ (NestJS)     │ (Remotion)   │             │
│  └──────────────┴──────────────┴──────────────┘             │
└────────────────┬────────────────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────────────────┐
│              Output Storage (S3 + PostgreSQL)               │
└─────────────────────────────────────────────────────────────┘
```

---

## Queue Types & Workers

### 1. Video Processing Queue

**Purpose:** Process videos (proxy generation, scene detection, transcription)

**Job Type:**
```typescript
interface VideoProcessingJob {
  id: string;
  intake_id: string;
  project_id: string;
  video_url: string;
  processing_type: 'proxy' | 'scene_detection' | 'transcription' | 'full';
  options: {
    scene_threshold?: number;
    min_scene_duration?: number;
    max_scenes?: number;
  };
}
```

**Worker Implementation (FastAPI):**
```python
from celery import Celery
from app.services.video_processor import VideoProcessor

celery_app = Celery('video_worker')
celery_app.conf.broker_url = 'redis://redis:6379'
celery_app.conf.result_backend = 'redis://redis:6379'

@celery_app.task(bind=True, max_retries=3)
def process_video(self, job_data):
    try:
        processor = VideoProcessor()
        
        # 1. Download video
        video = processor.download_video(job_data['video_url'])
        
        # 2. Generate proxy
        proxy = processor.generate_proxy(video)
        
        # 3. Detect scenes
        scenes = processor.detect_scenes(proxy, job_data['options'])
        
        # 4. Transcribe
        transcription = processor.transcribe(video)
        
        # 5. Upload results
        result_url = processor.upload_results({
            'scenes': scenes,
            'transcription': transcription
        })
        
        # 6. Update database
        processor.update_job_status(job_data['id'], 'completed', result_url)
        
        return {'status': 'completed', 'result_url': result_url}
        
    except Exception as exc:
        self.retry(exc=exc, countdown=60)
```

**Configuration:**
- **Workers:** 4-8 parallel workers
- **Max Retries:** 3
- **Timeout:** 3600 seconds (1 hour)
- **Priority:** Normal
- **Concurrency:** 4

---

### 2. Story Generation Queue

**Purpose:** Generate story packs using LangGraph + LLM

**Job Type:**
```typescript
interface StoryGenerationJob {
  id: string;
  project_id: string;
  topic: string;
  platforms: string[];
  style: string;
  target_audience: string;
  cta: string;
}
```

**Worker Implementation (NestJS):**
```typescript
import { Injectable } from '@nestjs/common';
import { Queue } from 'bull';
import { InjectQueue } from '@nestjs/bull';
import { StoryEngineService } from './story-engine.service';

@Injectable()
export class StoryGenerationWorker {
  constructor(
    @InjectQueue('story-generation') private queue: Queue,
    private storyEngine: StoryEngineService
  ) {}

  async processStoryGeneration(job: Job<StoryGenerationJob>) {
    try {
      const { topic, platforms, style, target_audience, cta } = job.data;

      // 1. Generate hook (2-3 sec)
      job.progress(10);
      const hook = await this.storyEngine.generateHook(topic, style);

      // 2. Generate core angle
      job.progress(20);
      const coreAngle = await this.storyEngine.generateCoreAngle(topic, hook);

      // 3. Generate scenes (6-8)
      job.progress(30);
      const scenes = await this.storyEngine.generateScenes(
        topic,
        coreAngle,
        8,
        style
      );

      // 4. Generate visuals in parallel
      job.progress(40);
      const visualPrompts = await Promise.all(
        scenes.map(scene => this.storyEngine.generateVisualPrompt(scene))
      );

      // 5. Generate voiceover
      job.progress(60);
      const voiceoverScript = await this.storyEngine.generateVoiceover(
        scenes,
        target_audience
      );

      // 6. Generate captions
      job.progress(75);
      const captions = await this.storyEngine.generateCaptions(
        voiceoverScript,
        'th'
      );

      // 7. Generate CTA
      job.progress(85);
      const ctaContent = await this.storyEngine.generateCTA(cta, platforms);

      // 8. Assemble stories
      job.progress(90);
      const stories = await this.storyEngine.assembleStories({
        hook,
        coreAngle,
        scenes,
        visualPrompts,
        voiceoverScript,
        captions,
        cta: ctaContent
      });

      // 9. Upload to S3
      job.progress(95);
      const uploadedStories = await this.storyEngine.uploadStories(stories);

      // 10. Update database
      job.progress(100);
      await this.storyEngine.updateGenerationStatus(
        job.data.id,
        'completed',
        uploadedStories
      );

      return { status: 'completed', stories: uploadedStories };

    } catch (error) {
      throw new Error(`Story generation failed: ${error.message}`);
    }
  }
}
```

**Configuration:**
- **Workers:** 2-4 parallel workers
- **Max Retries:** 2
- **Timeout:** 1800 seconds (30 minutes)
- **Priority:** High
- **Concurrency:** 2

---

### 3. Rendering Queue

**Purpose:** Render final videos using FFmpeg + Remotion

**Job Type:**
```typescript
interface RenderingJob {
  id: string;
  story_id: string;
  project_id: string;
  video_components: {
    scenes: Scene[];
    captions: Caption[];
    music?: string;
    transitions?: string;
  };
  output_format: 'mp4' | 'webm' | 'mov';
  quality: 'low' | 'medium' | 'high';
}
```

**Worker Implementation (Node.js + Remotion):**
```typescript
import { renderMedia } from '@remotion/renderer';
import { StoryVideo } from './compositions/StoryVideo';

export async function renderStory(job: Job<RenderingJob>) {
  try {
    const { story_id, video_components, output_format, quality } = job.data;

    // 1. Prepare composition
    job.progress(10);
    const composition = StoryVideo;

    // 2. Render video
    job.progress(20);
    const outputPath = `/tmp/story-${story_id}.${output_format}`;
    
    await renderMedia({
      composition,
      serveUrl: 'http://localhost:3000',
      codec: output_format === 'mp4' ? 'h264' : 'vp8',
      crf: quality === 'high' ? 18 : quality === 'medium' ? 23 : 28,
      outputLocation: outputPath,
      onProgress: (progress) => {
        job.progress(20 + progress * 70);
      }
    });

    // 3. Add audio (if needed)
    job.progress(90);
    if (video_components.music) {
      await addAudioToVideo(outputPath, video_components.music);
    }

    // 4. Upload to S3
    job.progress(95);
    const s3Url = await uploadToS3(outputPath, `stories/${story_id}`);

    // 5. Generate thumbnail
    job.progress(98);
    const thumbnailUrl = await generateThumbnail(outputPath);

    // 6. Update database
    job.progress(100);
    await updateStoryStatus(story_id, 'rendered', {
      video_url: s3Url,
      thumbnail_url: thumbnailUrl
    });

    return { status: 'completed', video_url: s3Url };

  } catch (error) {
    throw new Error(`Rendering failed: ${error.message}`);
  }
}
```

**Configuration:**
- **Workers:** 4-8 parallel workers
- **Max Retries:** 2
- **Timeout:** 3600 seconds (1 hour)
- **Priority:** Normal
- **Concurrency:** 4

---

### 4. Review Queue

**Purpose:** AI compliance check and human review

**Job Type:**
```typescript
interface ReviewJob {
  id: string;
  story_id: string;
  review_type: 'ai_review' | 'human_review';
  checks: {
    brand_guidelines?: boolean;
    legal_compliance?: boolean;
    platform_policies?: boolean;
    accessibility?: boolean;
  };
}
```

**Worker Implementation:**
```typescript
export async function reviewStory(job: Job<ReviewJob>) {
  try {
    const { story_id, review_type, checks } = job.data;

    if (review_type === 'ai_review') {
      // 1. Brand guidelines check
      job.progress(20);
      const brandCheck = await checkBrandGuidelines(story_id);

      // 2. Legal compliance check
      job.progress(40);
      const legalCheck = await checkLegalCompliance(story_id);

      // 3. Platform policy check
      job.progress(60);
      const platformCheck = await checkPlatformPolicies(story_id);

      // 4. Accessibility check
      job.progress(80);
      const accessibilityCheck = await checkAccessibility(story_id);

      // 5. Generate report
      job.progress(95);
      const report = {
        ai_score: calculateScore([brandCheck, legalCheck, platformCheck, accessibilityCheck]),
        issues: [
          ...brandCheck.issues,
          ...legalCheck.issues,
          ...platformCheck.issues,
          ...accessibilityCheck.issues
        ],
        recommendation: determineRecommendation([brandCheck, legalCheck, platformCheck, accessibilityCheck])
      };

      // 6. Save review
      job.progress(100);
      await saveReview(story_id, 'ai_review', report);

      return report;

    } else if (review_type === 'human_review') {
      // Queue for human reviewer
      await assignToHumanReviewer(story_id);
      return { status: 'awaiting_human_review' };
    }

  } catch (error) {
    throw new Error(`Review failed: ${error.message}`);
  }
}
```

**Configuration:**
- **Workers:** 1-2 parallel workers
- **Max Retries:** 1
- **Timeout:** 600 seconds (10 minutes)
- **Priority:** High
- **Concurrency:** 1

---

### 5. Release Queue

**Purpose:** Publish stories to platforms (YouTube, TikTok, Instagram, LinkedIn)

**Job Type:**
```typescript
interface ReleaseJob {
  id: string;
  release_id: string;
  story_ids: string[];
  platforms: string[];
  scheduled_for?: Date;
}
```

**Worker Implementation:**
```typescript
export async function releaseStory(job: Job<ReleaseJob>) {
  try {
    const { release_id, story_ids, platforms, scheduled_for } = job.data;

    // 1. Wait for scheduled time
    if (scheduled_for && new Date() < scheduled_for) {
      const delay = scheduled_for.getTime() - Date.now();
      await new Promise(resolve => setTimeout(resolve, delay));
    }

    // 2. Publish to each platform in parallel
    const publishPromises = platforms.map(async (platform) => {
      job.progress(10);

      if (platform === 'youtube') {
        return publishToYouTube(story_ids, release_id);
      } else if (platform === 'tiktok') {
        return publishToTikTok(story_ids, release_id);
      } else if (platform === 'instagram') {
        return publishToInstagram(story_ids, release_id);
      } else if (platform === 'linkedin') {
        return publishToLinkedIn(story_ids, release_id);
      }
    });

    const results = await Promise.all(publishPromises);

    // 3. Update release status
    job.progress(95);
    await updateReleaseStatus(release_id, 'released', results);

    // 4. Emit notification
    job.progress(100);
    await emitNotification({
      type: 'release_completed',
      release_id,
      platforms,
      story_count: story_ids.length
    });

    return { status: 'released', results };

  } catch (error) {
    throw new Error(`Release failed: ${error.message}`);
  }
}
```

**Configuration:**
- **Workers:** 2-4 parallel workers
- **Max Retries:** 2
- **Timeout:** 1200 seconds (20 minutes)
- **Priority:** Normal
- **Concurrency:** 2

---

## Job Flow Example: One-Click Story Generation

```
User submits topic
        ↓
API creates generation job
        ↓
Job queued in story-generation queue
        ↓
Worker picks up job
        ↓
LangGraph generates story structure
        ↓
Parallel generation:
├─ Visual prompts → Image generation
├─ Voiceover script → TTS
└─ Captions → Translation
        ↓
Remotion assembly
        ↓
FFmpeg rendering
        ↓
Upload to S3
        ↓
AI review (compliance check)
        ↓
Queue for human review (if needed)
        ↓
Approved → Ready for release
        ↓
Release to platforms
        ↓
Track analytics
```

---

## Error Handling & Retry Strategy

### Retry Policy

| Queue | Max Retries | Retry Delay | Backoff |
|-------|------------|-------------|---------|
| **video-processing** | 3 | 60s | Exponential |
| **story-generation** | 2 | 30s | Exponential |
| **rendering** | 2 | 60s | Exponential |
| **review** | 1 | 30s | Linear |
| **release** | 2 | 60s | Exponential |

### Dead Letter Queue

```typescript
// Jobs that fail after max retries go to DLQ
const dlq = new Queue('dlq', {
  connection: redis,
  defaultJobOptions: {
    attempts: 1
  }
});

// Process DLQ for investigation
dlq.process(async (job) => {
  // Log failure
  logger.error(`Job ${job.id} failed permanently`, {
    queue: job.data.queue,
    error: job.failedReason,
    data: job.data
  });

  // Alert team
  await alertTeam({
    type: 'job_failure',
    job_id: job.id,
    queue: job.data.queue
  });

  // Store for investigation
  await storeFailureReport(job);
});
```

---

## Monitoring & Observability

### Queue Metrics

```typescript
// Monitor queue depth
setInterval(async () => {
  const queues = [
    'video-processing',
    'story-generation',
    'rendering',
    'review',
    'release'
  ];

  for (const queueName of queues) {
    const queue = new Queue(queueName, { connection: redis });
    const counts = await queue.getJobCounts();

    logger.info(`Queue ${queueName}:`, {
      waiting: counts.waiting,
      active: counts.active,
      completed: counts.completed,
      failed: counts.failed
    });

    // Emit metrics
    metrics.gauge(`queue.${queueName}.waiting`, counts.waiting);
    metrics.gauge(`queue.${queueName}.active`, counts.active);
  }
}, 60000);
```

### Job Progress Tracking

```typescript
// WebSocket updates for real-time progress
job.on('progress', (progress) => {
  io.emit(`job:${job.id}:progress`, {
    progress,
    stage: job.data.current_stage,
    timestamp: new Date()
  });
});

job.on('completed', (result) => {
  io.emit(`job:${job.id}:completed`, {
    result,
    timestamp: new Date()
  });
});

job.on('failed', (error) => {
  io.emit(`job:${job.id}:failed`, {
    error: error.message,
    timestamp: new Date()
  });
});
```

---

## Scaling Recommendations

### Development

```yaml
video-processing: 1 worker
story-generation: 1 worker
rendering: 1 worker
review: 1 worker
release: 1 worker
```

### Production

```yaml
video-processing: 8 workers (CPU-intensive)
story-generation: 4 workers (LLM-intensive)
rendering: 8 workers (CPU-intensive)
review: 2 workers (I/O-bound)
release: 4 workers (I/O-bound)
```

### Auto-scaling Rules

- Scale up if queue depth > 10 for 5 minutes
- Scale down if queue depth < 2 for 10 minutes
- Maximum workers per queue: 16
- Minimum workers per queue: 1

---

**Document Version:** 1.0  
**Last Updated:** April 18, 2026  
**Status:** Ready for Implementation
