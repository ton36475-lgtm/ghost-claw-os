# Ghost Claw OS - Review, Release & Watch Logic

**Version:** 1.0  
**Status:** Design Phase  
**Author:** Manus AI  
**Date:** April 2026

---

## Overview

This document describes the three critical workflows:

1. **Review Gate**: AI compliance check → Human approval → Ready for release
2. **Release Center**: Publish to platforms (YouTube, TikTok, Instagram, LinkedIn)
3. **Watch Center**: Monitor performance & incidents

---

## 1. Review Gate Logic

### Workflow

```
Story Generated
    ↓
AI Compliance Review (Automated)
├─ Brand Guidelines Check
├─ Legal Compliance Check
├─ Platform Policy Check
├─ Accessibility Check
    ↓
AI Score: 0-100
├─ Score ≥ 85: Auto-approved → Ready for Release
├─ Score 70-84: Needs Human Review
└─ Score < 70: Rejected (needs rework)
    ↓
Human Review (if needed)
├─ Reviewer approves/rejects
├─ Adds comments/annotations
└─ Returns for revision if needed
    ↓
Approved → Release Queue
```

### AI Compliance Checks

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from pydantic import BaseModel

class ComplianceCheckResult(BaseModel):
    check_type: str
    passed: bool
    score: float  # 0-1
    issues: List[str]
    recommendations: List[str]

class ReviewGateService:
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4-turbo", temperature=0.3)
    
    async def check_brand_guidelines(self, story_id: str) -> ComplianceCheckResult:
        """
        Check if story aligns with brand guidelines
        - Logo placement
        - Color palette
        - Tone of voice
        - Brand messaging
        """
        
        story = get_story(story_id)
        brand_guidelines = get_brand_guidelines(story.project_id)
        
        prompt = ChatPromptTemplate.from_template("""
            Analyze this story against brand guidelines:
            
            Story:
            - Title: {title}
            - Description: {description}
            - Visual Concept: {visual_concept}
            - Voiceover Script: {voiceover}
            - On-screen Text: {on_screen_text}
            
            Brand Guidelines:
            - Logo: {logo_guidelines}
            - Colors: {color_palette}
            - Tone: {tone}
            - Messaging: {messaging}
            
            Check:
            1. Is the logo properly placed and visible?
            2. Are the colors consistent with brand palette?
            3. Is the tone consistent with brand voice?
            4. Are the messages aligned with brand values?
            
            Return JSON:
            {{
                "passed": true/false,
                "score": 0.95,
                "issues": ["Issue 1", "Issue 2"],
                "recommendations": ["Recommendation 1"]
            }}
        """)
        
        chain = prompt | self.llm
        
        result = await chain.ainvoke({
            'title': story.title,
            'description': story.description,
            'visual_concept': story.visual_concept,
            'voiceover': story.voiceover_script,
            'on_screen_text': story.on_screen_text,
            'logo_guidelines': brand_guidelines.logo,
            'color_palette': brand_guidelines.colors,
            'tone': brand_guidelines.tone,
            'messaging': brand_guidelines.messaging
        })
        
        return ComplianceCheckResult(**json.loads(result.content))
    
    async def check_legal_compliance(self, story_id: str) -> ComplianceCheckResult:
        """
        Check for legal/compliance issues
        - No misleading claims
        - Proper disclaimers
        - No prohibited content
        - Copyright compliance
        """
        
        story = get_story(story_id)
        
        prompt = ChatPromptTemplate.from_template("""
            Check this story for legal/compliance issues:
            
            Story Content:
            - Voiceover: {voiceover}
            - On-screen Text: {on_screen_text}
            - Visual Assets: {assets}
            
            Check:
            1. Are there any misleading or false claims?
            2. Are proper disclaimers included (if needed)?
            3. Is there any prohibited content (violence, hate speech, etc.)?
            4. Are all assets properly licensed/credited?
            5. Does it comply with platform policies?
            
            Return JSON:
            {{
                "passed": true/false,
                "score": 0.98,
                "issues": [],
                "recommendations": []
            }}
        """)
        
        chain = prompt | self.llm
        
        result = await chain.ainvoke({
            'voiceover': story.voiceover_script,
            'on_screen_text': story.on_screen_text,
            'assets': story.visual_assets
        })
        
        return ComplianceCheckResult(**json.loads(result.content))
    
    async def check_platform_policies(self, story_id: str, platforms: List[str]) -> ComplianceCheckResult:
        """
        Check compliance with platform-specific policies
        - YouTube: Community Guidelines
        - TikTok: Community Guidelines
        - Instagram: Community Standards
        - LinkedIn: Professional Standards
        """
        
        story = get_story(story_id)
        
        issues = []
        scores = []
        
        for platform in platforms:
            platform_policies = get_platform_policies(platform)
            
            prompt = ChatPromptTemplate.from_template(f"""
                Check this story against {platform} policies:
                
                Story: {{story_content}}
                Policies: {{policies}}
                
                Return JSON:
                {{
                    "passed": true/false,
                    "score": 0.95,
                    "issues": [],
                    "recommendations": []
                }}
            """)
            
            chain = prompt | self.llm
            
            result = await chain.ainvoke({
                'story_content': story.to_dict(),
                'policies': platform_policies
            })
            
            check_result = ComplianceCheckResult(**json.loads(result.content))
            scores.append(check_result.score)
            issues.extend(check_result.issues)
        
        return ComplianceCheckResult(
            check_type='platform_policies',
            passed=all(s >= 0.7 for s in scores),
            score=sum(scores) / len(scores),
            issues=issues,
            recommendations=[]
        )
    
    async def check_accessibility(self, story_id: str) -> ComplianceCheckResult:
        """
        Check accessibility compliance
        - Captions present and accurate
        - Color contrast sufficient
        - Text readable
        - Audio described (if needed)
        """
        
        story = get_story(story_id)
        
        prompt = ChatPromptTemplate.from_template("""
            Check this story for accessibility compliance:
            
            Story:
            - Has captions: {has_captions}
            - Caption accuracy: {caption_accuracy}
            - Text size: {text_size}
            - Color contrast: {color_contrast}
            - Audio description: {audio_description}
            
            Check:
            1. Are captions present and accurate?
            2. Is text size readable (minimum 16px)?
            3. Is color contrast sufficient (WCAG AA)?
            4. Is audio described for visual content?
            
            Return JSON:
            {{
                "passed": true/false,
                "score": 0.92,
                "issues": ["Issue 1"],
                "recommendations": ["Recommendation 1"]
            }}
        """)
        
        chain = prompt | self.llm
        
        result = await chain.ainvoke({
            'has_captions': bool(story.captions),
            'caption_accuracy': calculate_caption_accuracy(story),
            'text_size': story.text_size,
            'color_contrast': calculate_color_contrast(story),
            'audio_description': bool(story.audio_description)
        })
        
        return ComplianceCheckResult(**json.loads(result.content))

    async def run_full_review(self, story_id: str) -> dict:
        """Run all compliance checks and generate score"""
        
        checks = {
            'brand_guidelines': await self.check_brand_guidelines(story_id),
            'legal_compliance': await self.check_legal_compliance(story_id),
            'platform_policies': await self.check_platform_policies(story_id, ['youtube', 'tiktok', 'instagram']),
            'accessibility': await self.check_accessibility(story_id)
        }
        
        # Calculate overall score (weighted average)
        weights = {
            'brand_guidelines': 0.25,
            'legal_compliance': 0.35,
            'platform_policies': 0.25,
            'accessibility': 0.15
        }
        
        overall_score = sum(
            checks[key].score * weights[key]
            for key in checks
        )
        
        # Determine recommendation
        if overall_score >= 0.85:
            recommendation = 'auto_approved'
        elif overall_score >= 0.70:
            recommendation = 'needs_human_review'
        else:
            recommendation = 'rejected'
        
        # Store review in database
        review = Review(
            story_id=story_id,
            review_type='ai_review',
            status='completed',
            ai_score=overall_score,
            checks=checks,
            recommendation=recommendation,
            submitted_at=datetime.utcnow()
        )
        
        db.add(review)
        db.commit()
        
        # If auto-approved, move to release queue
        if recommendation == 'auto_approved':
            story = get_story(story_id)
            story.status = 'approved'
            db.commit()
            
            # Queue for release
            await queue_for_release(story_id)
        
        # If needs human review, notify reviewers
        elif recommendation == 'needs_human_review':
            await notify_reviewers(story_id, checks)
        
        return {
            'story_id': story_id,
            'overall_score': overall_score,
            'recommendation': recommendation,
            'checks': checks
        }
```

### Human Review Interface

```typescript
// API endpoint for human review
async function submitHumanReview(
  storyId: string,
  reviewData: {
    status: 'approved' | 'rejected' | 'needs_revision';
    comments: string;
    annotations: Annotation[];
  }
) {
  // 1. Validate review
  if (!reviewData.status) {
    throw new Error('Review status required');
  }

  // 2. Create review record
  const review = new Review({
    story_id: storyId,
    reviewer_id: getCurrentUserId(),
    review_type: 'human_review',
    status: reviewData.status,
    comments: reviewData.comments,
    submitted_at: new Date()
  });

  await db.save(review);

  // 3. Add annotations
  for (const annotation of reviewData.annotations) {
    const ann = new ReviewAnnotation({
      review_id: review.id,
      annotation_type: annotation.type,
      start_time: annotation.startTime,
      end_time: annotation.endTime,
      comment: annotation.comment,
      created_by: getCurrentUserId()
    });
    await db.save(ann);
  }

  // 4. Update story status
  const story = await getStory(storyId);
  
  if (reviewData.status === 'approved') {
    story.status = 'approved';
    await queueForRelease(storyId);
  } else if (reviewData.status === 'rejected') {
    story.status = 'rejected';
    await notifyCreator(storyId, 'Story rejected', reviewData.comments);
  } else if (reviewData.status === 'needs_revision') {
    story.status = 'needs_revision';
    await notifyCreator(storyId, 'Story needs revision', reviewData.comments);
  }

  await db.save(story);

  return review;
}
```

---

## 2. Release Center Logic

### Release Workflow

```
Approved Story
    ↓
Create Release
├─ Select stories
├─ Select platforms
├─ Schedule release time
├─ Add release notes
    ↓
Pre-release Checks
├─ Verify all assets uploaded
├─ Verify metadata complete
├─ Generate platform-specific formats
    ↓
Schedule Release
├─ Queue release job
├─ Set platform-specific settings
    ↓
Release Execution
├─ YouTube: Upload + Schedule
├─ TikTok: Upload + Schedule
├─ Instagram: Upload + Schedule
├─ LinkedIn: Upload + Schedule
    ↓
Post-release
├─ Track analytics
├─ Monitor comments/engagement
├─ Handle issues
```

### Release Implementation

```typescript
import { Queue } from 'bull';

class ReleaseService {
  constructor(
    private releaseQueue: Queue,
    private youtubeService: YouTubeService,
    private tiktokService: TikTokService,
    private instagramService: InstagramService,
    private linkedinService: LinkedInService
  ) {}

  async createRelease(releaseData: {
    project_id: string;
    story_ids: string[];
    platforms: string[];
    scheduled_for?: Date;
    release_notes?: string;
  }): Promise<Release> {
    // 1. Create release record
    const release = new Release({
      project_id: releaseData.project_id,
      release_name: `Release ${new Date().toISOString()}`,
      status: 'draft',
      scheduled_for: releaseData.scheduled_for,
      created_at: new Date()
    });

    await db.save(release);

    // 2. Add stories to release
    for (const storyId of releaseData.story_ids) {
      for (const platform of releaseData.platforms) {
        const releaseStory = new ReleaseStory({
          release_id: release.id,
          story_id: storyId,
          platform: platform
        });
        await db.save(releaseStory);
      }
    }

    return release;
  }

  async scheduleRelease(releaseId: string): Promise<void> {
    const release = await getRelease(releaseId);

    // 1. Validate all stories are approved
    const stories = await getReleaseStories(releaseId);
    for (const story of stories) {
      if (story.status !== 'approved') {
        throw new Error(`Story ${story.id} is not approved`);
      }
    }

    // 2. Prepare platform-specific formats
    for (const story of stories) {
      await this.prepareStoryForPlatforms(story, release.platforms);
    }

    // 3. Queue release job
    const releaseJob = await this.releaseQueue.add(
      'publish_release',
      {
        release_id: releaseId,
        stories: stories.map(s => s.id),
        platforms: release.platforms,
        scheduled_for: release.scheduled_for
      },
      {
        delay: release.scheduled_for
          ? release.scheduled_for.getTime() - Date.now()
          : 0,
        priority: 1
      }
    );

    // 4. Update release status
    release.status = 'scheduled';
    await db.save(release);
  }

  async publishRelease(releaseId: string): Promise<void> {
    const release = await getRelease(releaseId);
    const stories = await getReleaseStories(releaseId);

    // 1. Publish to each platform in parallel
    const publishPromises = release.platforms.map(async (platform) => {
      for (const story of stories) {
        if (platform === 'youtube') {
          await this.publishToYouTube(story, release);
        } else if (platform === 'tiktok') {
          await this.publishToTikTok(story, release);
        } else if (platform === 'instagram') {
          await this.publishToInstagram(story, release);
        } else if (platform === 'linkedin') {
          await this.publishToLinkedIn(story, release);
        }
      }
    });

    await Promise.all(publishPromises);

    // 2. Update release status
    release.status = 'released';
    release.released_at = new Date();
    await db.save(release);

    // 3. Emit notification
    await emitNotification({
      type: 'release_completed',
      release_id: releaseId,
      platforms: release.platforms,
      story_count: stories.length
    });
  }

  private async publishToYouTube(story: Story, release: Release): Promise<void> {
    const youtube = this.youtubeService;

    // 1. Prepare metadata
    const metadata = {
      title: story.title,
      description: story.description,
      tags: story.tags,
      categoryId: '22', // People & Blogs
      privacyStatus: 'public'
    };

    // 2. Upload video
    const videoId = await youtube.uploadVideo(
      story.video_url,
      metadata
    );

    // 3. Store publish status
    const publishStatus = new PlatformPublishStatus({
      release_id: release.id,
      platform: 'youtube',
      status: 'published',
      platform_url: `https://youtube.com/watch?v=${videoId}`,
      platform_id: videoId,
      published_at: new Date()
    });

    await db.save(publishStatus);
  }

  private async publishToTikTok(story: Story, release: Release): Promise<void> {
    const tiktok = this.tiktokService;

    // 1. Prepare video (convert to TikTok format if needed)
    const videoPath = await this.prepareVideoForTikTok(story.video_url);

    // 2. Upload video
    const videoId = await tiktok.uploadVideo(videoPath, {
      caption: story.title,
      hashtags: story.tags,
      allowComments: true,
      allowDuets: true,
      allowStitches: true
    });

    // 3. Store publish status
    const publishStatus = new PlatformPublishStatus({
      release_id: release.id,
      platform: 'tiktok',
      status: 'published',
      platform_url: `https://tiktok.com/@sirinx/video/${videoId}`,
      platform_id: videoId,
      published_at: new Date()
    });

    await db.save(publishStatus);
  }

  private async prepareVideoForTikTok(videoUrl: string): Promise<string> {
    // TikTok requires vertical video (9:16)
    // Convert if necessary
    const outputPath = `/tmp/tiktok-${Date.now()}.mp4`;

    const ffmpeg = new FFmpeg();
    ffmpeg.option('i', videoUrl);
    ffmpeg.filter('scale', 1080, 1920);
    ffmpeg.filter('pad', 1080, 1920, '(ow-iw)/2', '(oh-ih)/2', 'black');
    ffmpeg.option('c:v', 'libx264');
    ffmpeg.option('preset', 'fast');
    ffmpeg.output(outputPath);

    await ffmpeg.execute();

    return outputPath;
  }
}
```

---

## 3. Watch Center Logic

### Monitoring Dashboard

```typescript
class WatchCenterService {
  async getReleaseDashboard(releaseId: string): Promise<DashboardData> {
    const release = await getRelease(releaseId);
    const publishStatus = await getPlatformPublishStatus(releaseId);
    const analytics = await getReleaseAnalytics(releaseId);

    return {
      release: {
        id: release.id,
        name: release.release_name,
        status: release.status,
        released_at: release.released_at,
        story_count: release.story_count
      },
      platforms: publishStatus.map(ps => ({
        platform: ps.platform,
        status: ps.status,
        url: ps.platform_url,
        published_at: ps.published_at,
        error: ps.error_message
      })),
      analytics: {
        total_views: analytics.reduce((sum, a) => sum + a.views, 0),
        total_likes: analytics.reduce((sum, a) => sum + a.likes, 0),
        total_comments: analytics.reduce((sum, a) => sum + a.comments, 0),
        total_shares: analytics.reduce((sum, a) => sum + a.shares, 0),
        average_engagement_rate: calculateAverageEngagementRate(analytics),
        by_platform: groupAnalyticsByPlatform(analytics)
      },
      incidents: await getIncidents(releaseId)
    };
  }

  async trackAnalytics(releaseId: string): Promise<void> {
    const publishStatus = await getPlatformPublishStatus(releaseId);

    for (const ps of publishStatus) {
      if (ps.status !== 'published') continue;

      let analytics;

      if (ps.platform === 'youtube') {
        analytics = await this.getYouTubeAnalytics(ps.platform_id);
      } else if (ps.platform === 'tiktok') {
        analytics = await this.getTikTokAnalytics(ps.platform_id);
      } else if (ps.platform === 'instagram') {
        analytics = await this.getInstagramAnalytics(ps.platform_id);
      } else if (ps.platform === 'linkedin') {
        analytics = await this.getLinkedInAnalytics(ps.platform_id);
      }

      // Store analytics
      const analyticsRecord = new ReleaseAnalytics({
        release_id: releaseId,
        platform: ps.platform,
        views: analytics.views,
        likes: analytics.likes,
        comments: analytics.comments,
        shares: analytics.shares,
        click_through_rate: analytics.ctr,
        engagement_rate: analytics.engagement_rate,
        collected_at: new Date()
      });

      await db.save(analyticsRecord);
    }
  }

  async monitorIncidents(releaseId: string): Promise<void> {
    const publishStatus = await getPlatformPublishStatus(releaseId);

    for (const ps of publishStatus) {
      if (ps.status !== 'published') continue;

      // Check for common issues
      const issues = [];

      // 1. Check video availability
      if (!await this.isVideoAvailable(ps.platform, ps.platform_id)) {
        issues.push({
          type: 'video_unavailable',
          severity: 'critical',
          message: `Video is not available on ${ps.platform}`
        });
      }

      // 2. Check for copyright strikes
      if (await this.hasCopyrightStrike(ps.platform, ps.platform_id)) {
        issues.push({
          type: 'copyright_strike',
          severity: 'critical',
          message: `Copyright strike on ${ps.platform}`
        });
      }

      // 3. Check for community guideline violations
      if (await this.hasGuidelineViolation(ps.platform, ps.platform_id)) {
        issues.push({
          type: 'guideline_violation',
          severity: 'high',
          message: `Community guideline violation on ${ps.platform}`
        });
      }

      // 4. Check engagement metrics
      const analytics = await this.getLatestAnalytics(releaseId, ps.platform);
      if (analytics.engagement_rate < 0.01) {
        issues.push({
          type: 'low_engagement',
          severity: 'low',
          message: `Low engagement on ${ps.platform}`
        });
      }

      // Store incidents
      for (const issue of issues) {
        const incident = new SystemIncident({
          incident_type: issue.type,
          severity: issue.severity,
          title: `${issue.type} on ${ps.platform}`,
          description: issue.message,
          affected_resource: `${ps.platform}/${ps.platform_id}`,
          status: 'open',
          created_at: new Date()
        });

        await db.save(incident);

        // Alert team
        await this.alertTeam(incident);
      }
    }
  }

  async alertTeam(incident: SystemIncident): Promise<void> {
    // Send Slack notification
    await sendSlackNotification({
      channel: '#incidents',
      text: `🚨 ${incident.title}`,
      blocks: [
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: `*${incident.title}*\n${incident.description}`
          }
        },
        {
          type: 'context',
          elements: [
            {
              type: 'mrkdwn',
              text: `Severity: ${incident.severity}`
            }
          ]
        }
      ]
    });
  }
}
```

---

**Document Version:** 1.0  
**Last Updated:** April 18, 2026  
**Status:** Ready for Implementation
