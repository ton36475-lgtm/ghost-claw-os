# Ghost Claw OS - Product Summary

**Version:** 1.0  
**Status:** Production-Ready  
**Author:** Manus AI  
**Date:** April 2026

---

## Executive Overview

**Ghost Claw OS** is a **Media Production Operating System** for SIRINX that transforms content creation from days to minutes. It enables creators to go from topic/brief to multi-platform published stories in a single workflow.

### Value Proposition

| Before | After |
|--------|-------|
| 2-3 days to create 8 short-form videos | 4 minutes with One-Click Story Engine |
| Manual video editing for 2h+ content | Automated long-form autocut in 30 minutes |
| Manual asset management | Intelligent Google Drive integration |
| Manual compliance review | AI-powered review with human approval |
| Manual platform publishing | One-click multi-platform release |

---

## Core Problem

**Content creators waste 80% of time on non-creative tasks:**
- Searching for assets
- Editing long-form videos into short-form clips
- Writing scripts and prompts
- Compliance checking
- Platform-specific formatting
- Publishing and monitoring

**Ghost Claw OS solves this** by automating the entire production pipeline while maintaining human control over creative decisions.

---

## Solution Architecture

```
Input (Topic/Video)
    ↓
AI Processing (LangGraph + LLMs)
    ↓
Asset Integration (Google Drive)
    ↓
Rendering (FFmpeg + Remotion)
    ↓
Compliance Review (AI → Human)
    ↓
Multi-Platform Release (YouTube, TikTok, Instagram, LinkedIn)
    ↓
Performance Monitoring (Analytics + Incidents)
```

---

## 11 Core Modules

### 1. **Dashboard**
- Project overview
- Quick statistics
- Recent activity feed
- Quick actions (New project, One-Click, Long-form, Asset search)

### 2. **Projects**
- Project management
- Campaign organization
- Team collaboration
- Project settings

### 3. **Intake Studio**
- Brief/topic input
- Campaign metadata
- Target audience definition
- Platform selection

### 4. **One-Click Story Engine** ⭐
- **Input:** Single topic (e.g., "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้")
- **Output:** 6-8 short-form stories (6-8 seconds each)
- **Time:** 4 minutes
- **Features:**
  - Hook generation
  - Core angle extraction
  - Story pattern selection
  - Scene generation (6-8 scenes)
  - Visual prompt generation
  - Voiceover script generation
  - On-screen text generation
  - CTA generation
  - Thumbnail generation

### 5. **Long-form Autocut Studio** ⭐
- **Input:** 2h+ video file
- **Output:** 8-12 short-form stories (5-10 min shards)
- **Time:** 30 minutes
- **Pipeline:**
  - Proxy generation (fast preview)
  - Scene detection (PySceneDetect)
  - Selective transcription (Faster-Whisper)
  - Semantic ranking (LLM)
  - Key moment extraction
  - Short-form story assembly

### 6. **Asset Library**
- Google Drive integration
- Metadata indexing (Google Sheets)
- Smart search (tags, usage role, campaign)
- Asset preview & metadata
- Usage tracking

### 7. **Prompt Lab**
- Flow/Grok prompt generation
- Script generation
- Visual prompt generation
- Voiceover prompt generation
- CTA generation
- Thumbnail prompt generation

### 8. **Render Queue**
- Job management (FFmpeg + Remotion)
- Progress tracking
- Parallel processing
- Output management
- Error handling & retry

### 9. **Review Gate** ⭐
- AI compliance checks:
  - Brand guidelines alignment
  - Legal compliance
  - Platform policy compliance
  - Accessibility compliance
- AI scoring (0-100)
- Auto-approval (≥85) or human review
- Human review interface with annotations
- Approval workflow

### 10. **Release Center** ⭐
- Multi-platform publishing:
  - YouTube
  - TikTok
  - Instagram
  - LinkedIn
- Scheduling
- Release notes
- Platform-specific formatting
- Batch publishing

### 11. **Watch Center**
- Real-time analytics
- Platform performance tracking
- Incident monitoring
- Comment feed
- Engagement metrics
- Alert system

---

## Key Features

### One-Click Story Generation
```
Topic: "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้"
    ↓
LangGraph Pipeline:
  1. Hook generation
  2. Core angle extraction
  3. Story pattern selection
  4. 6-8 scenes generation
  5. Visual prompts
  6. Voiceover scripts
  7. On-screen text
  8. CTA + thumbnail
    ↓
Output: 8 complete short-form stories (6-8 sec each)
Time: 4 minutes
```

### Long-form Autocut
```
Input: 2h+ video
    ↓
1. Proxy generation (5 min)
2. Scene detection (5 min)
3. Transcription (10 min)
4. Semantic ranking (5 min)
5. Key moment extraction (3 min)
6. Story assembly (2 min)
    ↓
Output: 8-12 short-form stories
Time: 30 minutes
```

### Asset Memory
```
Google Drive: "Sirinx" folder (source of truth)
    ↓
Google Sheets: "SIRINX_Media_Asset_DB" (metadata)
    ↓
Search: tags + usage_role + campaign
    ↓
Output: preview + metadata + drive_url
```

### Review & Release
```
Story Generated
    ↓
AI Compliance Review (92/100)
    ↓
Auto-Approved (≥85) or Human Review
    ↓
Approved
    ↓
Multi-Platform Release:
  - YouTube: Upload + Schedule
  - TikTok: Upload + Schedule
  - Instagram: Upload + Schedule
  - LinkedIn: Upload + Schedule
    ↓
Watch Center: Monitor analytics & incidents
```

---

## Technical Stack

| Component | Technology |
|-----------|-----------|
| **Frontend** | Next.js 15 + React 19 + Tailwind CSS |
| **Backend** | FastAPI (Python) + NestJS (Node.js) |
| **Queue** | BullMQ + Redis |
| **AI/ML** | LangGraph + OpenAI/Grok hybrid |
| **Video** | FFmpeg + Remotion + PySceneDetect |
| **Database** | PostgreSQL 16 |
| **Storage** | S3-compatible (MinIO/AWS S3) |
| **Monitoring** | OpenTelemetry + PostHog |
| **Security** | Semgrep + Trivy + OAuth2 |
| **Deployment** | Vercel + AWS/Docker |

---

## Data Model

### Core Entities

```
Project
├── Campaign
├── Intake (brief/topic/video)
├── Story (generated short-form)
├── Release (multi-platform batch)
├── Review (compliance check)
├── RenderJob (video processing)
└── Analytics (performance tracking)

Asset
├── Google Drive metadata
├── Usage tracking
└── Campaign association

User
├── Role (creator, reviewer, admin)
├── Permissions
└── Team membership
```

---

## Compliance & Security

✅ **Compliance-First**
- AI review before human review
- All content flagged for potential issues
- Audit trail for all decisions

✅ **Sandbox-First**
- All processing isolated
- No direct platform API access
- Staged release workflow

✅ **Least Privilege**
- Role-based access control
- Project-level permissions
- Audit logging

✅ **Untrusted Inputs**
- All inputs validated
- All outputs sanitized
- No SQL injection, XSS, or CSRF vulnerabilities

✅ **Prohibited Activities**
- ❌ No gambling
- ❌ No stealth/spoofing/proxy evasion
- ❌ No multi-account evasion
- ❌ No identity-card processing
- ❌ No credential storage
- ❌ No platform-violating automation

---

## Performance Targets

| Metric | Target |
|--------|--------|
| One-Click story generation | < 5 minutes |
| Long-form autocut | < 30 minutes |
| API response time | < 500ms |
| Database query time | < 100ms |
| Page load time | < 2 seconds |
| Uptime | > 99.9% |
| Test coverage | > 80% |

---

## Success Metrics

### User Metrics
- Time to first story: < 10 minutes
- User satisfaction: > 4.5/5
- Feature adoption: > 80%
- Retention rate: > 70%

### Business Metrics
- Stories created per user per week: > 10
- Multi-platform releases per week: > 5
- Average engagement rate: > 5%
- Cost per story: < $1

### Technical Metrics
- System uptime: > 99.9%
- Error rate: < 0.1%
- Average response time: < 500ms
- Queue depth: < 100 jobs

---

## Sample Project

### Project: "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้"

**Scenario:** Educational series about unpredictable SME electricity costs

**One-Click Generation:**
```
Input:
- Topic: "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้"
- Platforms: YouTube, TikTok, Instagram
- Style: Educational
- Target: SME business owners
- CTA: "Subscribe for more SME tips"

Output (4 minutes):
1. Story 1: Hook - "Why SME electricity costs vary"
2. Story 2: Problem - "Unpredictable cost factors"
3. Story 3: Solution - "How to manage costs"
4. Story 4: CTA - "Subscribe for more"
5. Story 5: Hook (variant)
6. Story 6: Problem (variant)
7. Story 7: Solution (variant)
8. Story 8: CTA (variant)

Each story: 6-8 seconds, complete with:
- Visual concept
- Voiceover script
- On-screen text
- Camera angles
- Image prompts
- Video prompts
- Thumbnail
```

**Long-form Autocut:**
```
Input:
- Video: sme-electricity-2h.mp4 (2 hours)
- Topic: "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้"

Output (30 minutes):
- 8-12 short-form stories extracted
- Each story: 5-10 minutes of original content
- Automatically edited with:
  - Scene transitions
  - B-roll integration
  - Captions
  - Music
  - Sound effects
```

**Review & Release:**
```
AI Review:
- Brand guidelines: ✓ 95/100
- Legal compliance: ✓ 98/100
- Platform policies: ✓ 88/100
- Accessibility: ✓ 92/100
- Overall: ✓ 92/100 (Auto-approved)

Release:
- YouTube: Scheduled for 2026-04-20 10:00 AM
- TikTok: Scheduled for 2026-04-20 10:00 AM
- Instagram: Scheduled for 2026-04-20 10:00 AM
- LinkedIn: Scheduled for 2026-04-20 10:00 AM

Analytics (24 hours later):
- Total views: 125.4K
- YouTube: 45.2K views, 8.5% engagement
- TikTok: 65.1K views, 13.8% engagement
- Instagram: 15.1K views, 11.3% engagement
```

---

## Roadmap

### V1 (Weeks 1-12)
- ✅ Dashboard
- ✅ One-Click Story Engine
- ✅ Long-form Autocut Studio
- ✅ Asset Library
- ✅ Prompt Lab
- ✅ Render Queue
- ✅ Review Gate
- ✅ Release Center
- ✅ Watch Center

### V1.1 (Weeks 13-16)
- Collaboration features
- Advanced analytics
- API marketplace
- Mobile app

### V2 (Weeks 17+)
- AI personalization
- Advanced workflows
- Enterprise features
- Custom integrations

---

## Business Model

### Pricing Tiers

| Tier | Price | Stories/Month | Features |
|------|-------|---------------|----------|
| **Starter** | $99/mo | 50 | One-Click, Asset Library |
| **Pro** | $299/mo | 500 | + Long-form, Prompt Lab |
| **Enterprise** | Custom | Unlimited | + API, Custom integrations |

### Revenue Streams
1. **Subscription** (primary)
2. **API usage** (secondary)
3. **Professional services** (tertiary)

---

## Competitive Advantage

1. **Speed** - 4 minutes vs. 2-3 days
2. **Automation** - Long-form to short-form in 30 minutes
3. **Integration** - Google Drive + LLM + Video in one platform
4. **Compliance** - AI review before human review
5. **Multi-platform** - One-click publishing to 4+ platforms

---

## Risk Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|-----------|
| LLM API rate limits | High | High | Caching, fallback models |
| Video processing delays | Medium | High | Parallel processing, scaling |
| Platform API changes | Medium | Medium | Abstraction layer, monitoring |
| Security vulnerabilities | Low | Critical | Regular audits, penetration testing |

---

## Next Steps

1. **Review this summary** with stakeholders
2. **Approve architecture** (see 03-SYSTEM-ARCHITECTURE.md)
3. **Allocate team** (see 13-INITIAL-BUILD-PLAN.md)
4. **Start Phase 1** (Foundation & Infrastructure)
5. **Build sample project** (ค่าไฟธุรกิจ SME)

---

**Document Version:** 1.0  
**Last Updated:** April 18, 2026  
**Status:** Ready for Implementation
