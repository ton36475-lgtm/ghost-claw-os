# Ghost Claw OS - UI Screens & Design Specifications

**Version:** 1.0  
**Status:** Design Phase  
**Author:** Manus AI  
**Date:** April 2026

---

## Design System

### Color Palette

| Name | Light | Dark | Usage |
|------|-------|------|-------|
| **Primary** | #0066FF | #3399FF | Buttons, links, highlights |
| **Secondary** | #6B21A8 | #A855F7 | Accents, secondary actions |
| **Success** | #10B981 | #34D399 | Success states, approvals |
| **Warning** | #F59E0B | #FBBF24 | Warnings, cautions |
| **Error** | #EF4444 | #F87171 | Errors, rejections |
| **Background** | #FFFFFF | #0F172A | Page background |
| **Surface** | #F3F4F6 | #1E293B | Card background |
| **Text** | #111827 | #F1F5F9 | Primary text |
| **Muted** | #6B7280 | #94A3B8 | Secondary text |

### Typography

```css
/* Headings */
h1: 32px, 700, line-height: 1.2
h2: 24px, 700, line-height: 1.3
h3: 20px, 600, line-height: 1.4
h4: 16px, 600, line-height: 1.5

/* Body */
body: 14px, 400, line-height: 1.6
small: 12px, 400, line-height: 1.5

/* Font Family */
Font: Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif
```

### Spacing

```css
/* 8px grid system */
xs: 4px
sm: 8px
md: 16px
lg: 24px
xl: 32px
2xl: 48px
3xl: 64px
```

---

## Screen Layouts

### 1. Dashboard

**Purpose:** Overview of projects, recent activity, quick actions  
**URL:** `/dashboard`

```
┌─────────────────────────────────────────────────────────────┐
│ Ghost Claw OS                                    [User Menu] │
├─────────────────────────────────────────────────────────────┤
│ Dashboard                                                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────┐ │
│  │ Quick Actions    │  │ Recent Projects  │  │ Statistics │ │
│  │                  │  │                  │  │            │ │
│  │ [+ New Project]  │  │ • Project 1      │  │ Stories: 45│ │
│  │ [+ One-Click]    │  │ • Project 2      │  │ Views: 12k │ │
│  │ [+ Long-form]    │  │ • Project 3      │  │ Engagement:│ │
│  │ [+ Asset Search] │  │ [View All]       │  │ 8.5%       │ │
│  │                  │  │                  │  │            │ │
│  └──────────────────┘  └──────────────────┘  └────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Recent Activity                                        │ │
│  │                                                        │ │
│  │ • Story "Hook" approved by John (2 hours ago)        │ │
│  │ • Release "Q1-2026-SME" published (4 hours ago)      │ │
│  │ • 8 stories generated from "Electricity Costs" (1d)  │ │
│  │ • Video uploaded: sme-electricity-2h.mp4 (2d)        │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Components:**
- Header with navigation
- Quick actions panel
- Recent projects list
- Statistics cards
- Activity feed
- Sidebar navigation

---

### 2. One-Click Story Engine

**Purpose:** Generate stories from a single topic  
**URL:** `/modules/story-engine`

```
┌─────────────────────────────────────────────────────────────┐
│ Ghost Claw OS > Story Engine                   [User Menu]   │
├─────────────────────────────────────────────────────────────┤
│ One-Click Story Engine                                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Generate Stories from Topic                         │  │
│  │                                                      │  │
│  │ Topic *                                              │  │
│  │ ┌──────────────────────────────────────────────────┐ │  │
│  │ │ ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้                 │ │  │
│  │ └──────────────────────────────────────────────────┘ │  │
│  │                                                      │  │
│  │ Platforms *                                          │  │
│  │ ☑ YouTube  ☑ TikTok  ☑ Instagram  ☐ LinkedIn       │  │
│  │                                                      │  │
│  │ Style                                                │  │
│  │ ◉ Educational  ○ Entertaining  ○ Promotional       │  │
│  │                                                      │  │
│  │ Target Audience                                      │  │
│  │ ┌──────────────────────────────────────────────────┐ │  │
│  │ │ SME business owners, 25-55 years old            │ │  │
│  │ └──────────────────────────────────────────────────┘ │  │
│  │                                                      │  │
│  │ Call-to-Action                                       │  │
│  │ ┌──────────────────────────────────────────────────┐ │  │
│  │ │ Subscribe for more SME tips                     │ │  │
│  │ └──────────────────────────────────────────────────┘ │  │
│  │                                                      │  │
│  │ [Generate Stories] (Estimated: 4 minutes)           │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Generation Progress                                  │  │
│  │                                                      │  │
│  │ Generating hook...                                   │  │
│  │ ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  45%│  │
│  │                                                      │  │
│  │ Estimated time remaining: 2 minutes 15 seconds      │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Components:**
- Topic input field
- Platform selection checkboxes
- Style selection radio buttons
- Target audience textarea
- CTA input field
- Generate button with estimated time
- Progress bar with current stage
- Real-time status updates

---

### 3. Asset Library

**Purpose:** Search and browse media assets  
**URL:** `/modules/asset-library`

```
┌─────────────────────────────────────────────────────────────┐
│ Ghost Claw OS > Asset Library                  [User Menu]   │
├─────────────────────────────────────────────────────────────┤
│ Asset Library                                                │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Search Assets                                        │  │
│  │                                                      │  │
│  │ Search by tags (comma-separated)                     │  │
│  │ ┌──────────────────────────────────────────────────┐ │  │
│  │ │ electricity, business, chart                     │ │  │
│  │ └──────────────────────────────────────────────────┘ │  │
│  │                                                      │  │
│  │ Usage Role                                           │  │
│  │ [All ▼] [Hero] [Supporting] [Transition]            │  │
│  │                                                      │  │
│  │ Campaign                                             │  │
│  │ [Q1-2026-SME ▼]                                     │  │
│  │                                                      │  │
│  │ Asset Type                                           │  │
│  │ [All ▼]  [Image] [Video] [Audio] [Template]         │  │
│  │                                                      │  │
│  │ [Search]                                             │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  Results: 245 assets                                        │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ [Image]  │  │ [Image]  │  │ [Image]  │  │ [Image]  │   │
│  │ SME-1    │  │ SME-2    │  │ SME-3    │  │ SME-4    │   │
│  │ Hero     │  │ Hero     │  │ Support  │  │ Support  │   │
│  │ 12 uses  │  │ 8 uses   │  │ 5 uses   │  │ 3 uses   │   │
│  │ [Select] │  │ [Select] │  │ [Select] │  │ [Select] │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│                                                              │
│  [Load More]                                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Components:**
- Search filters (tags, role, campaign, type)
- Asset grid with thumbnails
- Asset metadata (type, role, usage count)
- Selection checkboxes
- Pagination/infinite scroll
- Preview modal on click

---

### 4. Render Queue

**Purpose:** Monitor and manage video rendering jobs  
**URL:** `/modules/render-queue`

```
┌─────────────────────────────────────────────────────────────┐
│ Ghost Claw OS > Render Queue                   [User Menu]   │
├─────────────────────────────────────────────────────────────┤
│ Render Queue                                                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Active Jobs: 3  |  Queued: 12  |  Completed: 145           │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Job ID      │ Story           │ Status    │ Progress  │ │
│  ├────────────────────────────────────────────────────────┤ │
│  │ job-001     │ Story 1: Hook   │ ▶ Active  │ ███░░░░░░│ │
│  │             │                 │           │ 30%       │ │
│  ├────────────────────────────────────────────────────────┤ │
│  │ job-002     │ Story 2: Problem│ ▶ Active  │ ██░░░░░░░│ │
│  │             │                 │           │ 20%       │ │
│  ├────────────────────────────────────────────────────────┤ │
│  │ job-003     │ Story 3: Solution│ ▶ Active │ █░░░░░░░░│ │
│  │             │                 │           │ 10%       │ │
│  ├────────────────────────────────────────────────────────┤ │
│  │ job-004     │ Story 4: CTA    │ ⏳ Queued │ 0%        │ │
│  ├────────────────────────────────────────────────────────┤ │
│  │ job-005     │ Story 5: Hook   │ ⏳ Queued │ 0%        │ │
│  ├────────────────────────────────────────────────────────┤ │
│  │ job-006     │ Story 6: Problem│ ✓ Done    │ 100%      │ │
│  │             │                 │           │ 4m 23s    │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  [Pause All] [Resume All] [Clear Completed]                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Components:**
- Job statistics (active, queued, completed)
- Job list with status indicators
- Progress bars
- Action buttons (pause, resume, cancel)
- Real-time updates via WebSocket

---

### 5. Review Gate

**Purpose:** AI compliance check and human approval  
**URL:** `/modules/review-gate`

```
┌─────────────────────────────────────────────────────────────┐
│ Ghost Claw OS > Review Gate                    [User Menu]   │
├─────────────────────────────────────────────────────────────┤
│ Review Gate                                                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Pending Review: 5  |  Approved: 32  |  Rejected: 2         │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Story: "Hook - Why SME electricity costs vary"       │  │
│  │                                                      │  │
│  │ AI Compliance Review                                 │  │
│  │ Overall Score: 92/100 ✓ AUTO-APPROVED               │  │
│  │                                                      │  │
│  │ ✓ Brand Guidelines      95/100                       │  │
│  │ ✓ Legal Compliance      98/100                       │  │
│  │ ✓ Platform Policies     88/100                       │  │
│  │ ✓ Accessibility         92/100                       │  │
│  │                                                      │  │
│  │ Issues Found:                                        │  │
│  │ ⚠ Color contrast on subtitle could be improved      │  │
│  │ ⚠ Consider adding audio description                 │  │
│  │                                                      │  │
│  │ Recommendations:                                     │  │
│  │ • Increase subtitle font size by 2px                │  │
│  │ • Add audio description for visual elements         │  │
│  │                                                      │  │
│  │ [Approve] [Request Changes] [Reject]                │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Video Preview                                        │  │
│  │                                                      │  │
│  │ ┌──────────────────────────────────────────────────┐ │  │
│  │ │                                                  │ │  │
│  │ │           [Video Player]                        │ │  │
│  │ │                                                  │ │  │
│  │ │ ▶ ────────────────────────────── 0:08           │ │  │
│  │ │                                                  │ │  │
│  │ └──────────────────────────────────────────────────┘ │  │
│  │                                                      │  │
│  │ [Add Annotation] [View Captions]                    │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Components:**
- Story preview with video player
- AI compliance check results
- Issue list with severity indicators
- Recommendations
- Approval/rejection buttons
- Video annotation tools
- Caption viewer

---

### 6. Release Center

**Purpose:** Publish stories to multiple platforms  
**URL:** `/modules/release-center`

```
┌─────────────────────────────────────────────────────────────┐
│ Ghost Claw OS > Release Center                 [User Menu]   │
├─────────────────────────────────────────────────────────────┤
│ Release Center                                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Create Release                                       │  │
│  │                                                      │  │
│  │ Release Name                                         │  │
│  │ ┌──────────────────────────────────────────────────┐ │  │
│  │ │ Q1-2026-SME-Electricity-Release-1               │ │  │
│  │ └──────────────────────────────────────────────────┘ │  │
│  │                                                      │  │
│  │ Stories (8 selected)                                 │  │
│  │ ☑ Story 1: Hook                                     │  │
│  │ ☑ Story 2: Problem                                  │  │
│  │ ☑ Story 3: Solution                                 │  │
│  │ ☑ Story 4: CTA                                      │  │
│  │ [Show All 8 Stories]                                │  │
│  │                                                      │  │
│  │ Platforms                                            │  │
│  │ ☑ YouTube  ☑ TikTok  ☑ Instagram  ☐ LinkedIn       │  │
│  │                                                      │  │
│  │ Schedule Release                                     │  │
│  │ ◉ Publish Now  ○ Schedule for later                │  │
│  │                                                      │  │
│  │ Release Notes                                        │  │
│  │ ┌──────────────────────────────────────────────────┐ │  │
│  │ │ Educational series about SME electricity costs  │ │  │
│  │ └──────────────────────────────────────────────────┘ │  │
│  │                                                      │  │
│  │ [Create Release] [Preview]                           │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Recent Releases                                      │  │
│  │                                                      │  │
│  │ Release: Q1-2026-SME-Release-1                       │  │
│  │ Status: ✓ Published (2 hours ago)                    │  │
│  │ Stories: 8  |  Platforms: 3                          │  │
│  │ Views: 5.2k  |  Engagement: 8.5%                    │  │
│  │ [View Analytics]                                     │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Components:**
- Release creation form
- Story selection checkboxes
- Platform selection
- Schedule picker
- Release notes textarea
- Recent releases list
- Analytics summary

---

### 7. Watch Center

**Purpose:** Monitor performance and incidents  
**URL:** `/modules/watch-center`

```
┌─────────────────────────────────────────────────────────────┐
│ Ghost Claw OS > Watch Center                   [User Menu]   │
├─────────────────────────────────────────────────────────────┤
│ Watch Center                                                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Total Views  │  │ Engagement   │  │ Incidents    │      │
│  │ 125.4K       │  │ 8.5%         │  │ 0 Critical   │      │
│  │ ↑ 12% (7d)   │  │ ↑ 2.1% (7d)  │  │ 2 Warnings   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Platform Performance (Last 7 Days)                    │ │
│  │                                                        │ │
│  │ YouTube:     45.2K views  │ ████████████░░░░░░░░░░░  │ │
│  │ TikTok:      65.1K views  │ ██████████████░░░░░░░░░░ │ │
│  │ Instagram:   15.1K views  │ ████░░░░░░░░░░░░░░░░░░░░ │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Incidents & Alerts                                    │ │
│  │                                                        │ │
│  │ ⚠ Story "Hook" - Low engagement on Instagram         │ │
│  │   Engagement: 2.1% (Expected: 5%)                    │ │
│  │   [Investigate] [Dismiss]                            │ │
│  │                                                        │ │
│  │ ⚠ Release "Q1-SME-1" - Copyright claim on YouTube    │ │
│  │   Status: Under review                               │ │
│  │   [View Details] [Appeal]                            │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Recent Comments & Engagement                          │ │
│  │                                                        │ │
│  │ YouTube - Story 1: Hook                              │ │
│  │ "Great explanation! Very helpful for my business"   │ │
│  │ ❤ 45  💬 12  ↗ 3                                     │ │
│  │                                                        │ │
│  │ TikTok - Story 2: Problem                            │ │
│  │ "I had no idea about this! 😱"                       │ │
│  │ ❤ 234  💬 89  ↗ 45                                   │ │
│  │                                                        │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Components:**
- Key metrics cards
- Platform performance chart
- Incident alerts
- Comments feed
- Real-time updates
- Drill-down analytics

---

## Component Library

### Buttons

```tsx
// Primary Button
<Button variant="primary" size="md">
  Generate Stories
</Button>

// Secondary Button
<Button variant="secondary" size="md">
  Cancel
</Button>

// Danger Button
<Button variant="danger" size="md">
  Delete
</Button>

// Loading State
<Button variant="primary" isLoading>
  Processing...
</Button>

// Disabled State
<Button variant="primary" disabled>
  Disabled
</Button>
```

### Input Fields

```tsx
// Text Input
<Input
  label="Topic"
  placeholder="Enter topic"
  value={topic}
  onChange={(e) => setTopic(e.target.value)}
  error={error}
/>

// Select Dropdown
<Select
  label="Platform"
  options={[
    { value: 'youtube', label: 'YouTube' },
    { value: 'tiktok', label: 'TikTok' }
  ]}
  value={platform}
  onChange={(e) => setPlatform(e.target.value)}
/>

// Textarea
<Textarea
  label="Description"
  placeholder="Enter description"
  value={description}
  onChange={(e) => setDescription(e.target.value)}
  rows={4}
/>
```

### Cards

```tsx
// Story Card
<StoryCard
  title="Story 1: Hook"
  thumbnail="https://..."
  duration={8}
  status="draft"
  onSelect={() => handleSelect()}
/>

// Metric Card
<MetricCard
  title="Total Views"
  value="125.4K"
  change="+12%"
  trend="up"
/>

// Asset Card
<AssetCard
  name="SME-Electricity-Hero-1"
  type="image"
  thumbnail="https://..."
  usageCount={12}
  onSelect={() => handleSelect()}
/>
```

---

## Responsive Design

### Breakpoints

```css
/* Mobile First */
xs: 0px
sm: 640px
md: 768px
lg: 1024px
xl: 1280px
2xl: 1536px
```

### Mobile Optimizations

- **Touch targets**: Minimum 44px × 44px
- **Font sizes**: Minimum 16px for inputs (prevent zoom on iOS)
- **Spacing**: Increased padding on mobile
- **Navigation**: Bottom tab bar or hamburger menu
- **Forms**: Single column layout

---

**Document Version:** 1.0  
**Last Updated:** April 18, 2026  
**Status:** Ready for Implementation
