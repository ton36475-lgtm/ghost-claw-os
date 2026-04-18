# Ghost Claw OS - Asset Memory Logic

**Version:** 1.0  
**Status:** Design Phase  
**Author:** Manus AI  
**Date:** April 2026

---

## Overview

The Asset Memory system provides semantic search and management of media assets stored in Google Drive, enabling creators to find and reuse the perfect visual, audio, or template for any story.

**Source of Truth:** Google Drive folder "Sirinx"  
**Metadata Index:** Google Sheet "SIRINX_Media_Asset_DB"  
**Search Capability:** Tags, usage role, campaign/scene  
**Response:** Preview + metadata + direct link

---

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                   Google Drive                               │
│  Sirinx/
│  ├── Images/
│  ├── Videos/
│  ├── Audio/
│  └── Templates/
└────────────────┬─────────────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────────────┐
│           Google Sheets (Metadata Index)                     │
│  SIRINX_Media_Asset_DB
│  ├── asset_id
│  ├── asset_name
│  ├── asset_type
│  ├── tags
│  ├── usage_role
│  ├── campaign_or_scene
│  ├── drive_url
│  ├── preview_url
│  ├── usage_count
│  └── last_used
└────────────────┬─────────────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────────────┐
│          PostgreSQL (Local Cache)                            │
│  assets table
│  asset_tags table
│  asset_usage table
└────────────────┬─────────────────────────────────────────────┘
                 ↓
┌──────────────────────────────────────────────────────────────┐
│            Asset Library API                                 │
│  - Search by tags
│  - Search by usage_role
│  - Search by campaign
│  - Get asset details
│  - Track usage
└──────────────────────────────────────────────────────────────┘
```

---

## Data Model

### Asset Structure

```typescript
interface Asset {
  id: string;
  name: string;
  type: 'image' | 'video' | 'audio' | 'template';
  drive_url: string;
  preview_url: string;
  usage_role: 'hero' | 'supporting' | 'transition' | 'effect' | 'music' | 'voiceover';
  campaign_or_scene: string;  // e.g., 'Q1-2026-SME', 'electricity-hero'
  tags: string[];  // e.g., ['electricity', 'business', 'sme', 'chart']
  metadata: {
    width?: number;
    height?: number;
    duration?: number;
    format: string;
    size: number;
    created_date: string;
  };
  usage_count: number;
  last_used: Date;
  created_at: Date;
  updated_at: Date;
}
```

### Google Sheet Schema

| Column | Type | Example |
|--------|------|---------|
| `asset_id` | UUID | `550e8400-e29b-41d4-a716-446655440000` |
| `asset_name` | String | `SME-Electricity-Hero-1.jpg` |
| `asset_type` | Enum | `image` |
| `tags` | Array | `electricity,business,sme,chart` |
| `usage_role` | String | `hero` |
| `campaign_or_scene` | String | `Q1-2026-SME` |
| `drive_url` | URL | `https://drive.google.com/file/d/...` |
| `preview_url` | URL | `https://cdn.example.com/preview.jpg` |
| `usage_count` | Integer | `5` |
| `last_used` | DateTime | `2026-04-15T14:30:00Z` |

---

## Implementation

### 1. Google Drive Integration

```python
from google.oauth2.service_account import Credentials
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload
import os

class GoogleDriveManager:
    def __init__(self):
        self.creds = Credentials.from_service_account_file(
            os.getenv('GOOGLE_SERVICE_ACCOUNT_JSON')
        )
        self.drive_service = build('drive', 'v3', credentials=self.creds)
        self.sheets_service = build('sheets', 'v4', credentials=self.creds)
        self.sirinx_folder_id = os.getenv('GOOGLE_DRIVE_FOLDER_ID')
        self.sheet_id = os.getenv('GOOGLE_SHEETS_ID')

    async def list_assets_from_drive(self) -> List[dict]:
        """List all assets from Sirinx folder"""
        
        try:
            results = self.drive_service.files().list(
                q=f"'{self.sirinx_folder_id}' in parents and trashed=false",
                spaces='drive',
                fields='files(id, name, mimeType, webViewLink, createdTime, size)',
                pageSize=1000,
                supportsAllDrives=True
            ).execute()

            files = results.get('files', [])
            
            assets = []
            for file in files:
                asset = {
                    'drive_id': file['id'],
                    'name': file['name'],
                    'mime_type': file['mimeType'],
                    'drive_url': file['webViewLink'],
                    'created_date': file['createdTime'],
                    'size': file.get('size', 0),
                    'type': self._determine_asset_type(file['mimeType'])
                }
                assets.append(asset)
            
            return assets

        except Exception as e:
            logger.error(f"Error listing assets from Drive: {e}")
            raise

    def _determine_asset_type(self, mime_type: str) -> str:
        """Determine asset type from MIME type"""
        
        if mime_type.startswith('image/'):
            return 'image'
        elif mime_type.startswith('video/'):
            return 'video'
        elif mime_type.startswith('audio/'):
            return 'audio'
        else:
            return 'template'

    async def get_preview_url(self, drive_id: str) -> str:
        """Generate preview URL for asset"""
        
        # For images and videos, use Google Drive preview
        return f"https://drive.google.com/thumbnail?id={drive_id}&sz=w200"

    async def download_asset(self, drive_id: str, output_path: str):
        """Download asset from Google Drive"""
        
        request = self.drive_service.files().get_media(fileId=drive_id)
        
        with open(output_path, 'wb') as f:
            downloader = MediaIoBaseDownload(f, request)
            done = False
            while not done:
                status, done = downloader.next_chunk()
```

### 2. Metadata Sync

```python
async def sync_assets_from_google_sheets():
    """Sync asset metadata from Google Sheets to PostgreSQL"""
    
    drive_manager = GoogleDriveManager()
    
    try:
        # 1. Read metadata from Google Sheet
        sheet_result = drive_manager.sheets_service.spreadsheets().values().get(
            spreadsheetId=drive_manager.sheet_id,
            range='SIRINX_Media_Asset_DB!A:J'
        ).execute()

        rows = sheet_result.get('values', [])[1:]  # Skip header
        
        # 2. Process each row
        for row in rows:
            if len(row) < 9:
                continue
            
            asset_id, asset_name, asset_type, tags, usage_role, campaign, drive_url, preview_url, usage_count = row[:9]
            
            # 3. Check if asset exists in database
            existing_asset = db.query(Asset).filter(
                Asset.drive_url == drive_url
            ).first()
            
            if existing_asset:
                # Update existing asset
                existing_asset.name = asset_name
                existing_asset.type = asset_type
                existing_asset.usage_role = usage_role
                existing_asset.campaign_or_scene = campaign
                existing_asset.usage_count = int(usage_count) if usage_count else 0
                existing_asset.updated_at = datetime.utcnow()
            else:
                # Create new asset
                new_asset = Asset(
                    name=asset_name,
                    type=asset_type,
                    drive_url=drive_url,
                    preview_url=preview_url,
                    usage_role=usage_role,
                    campaign_or_scene=campaign,
                    usage_count=int(usage_count) if usage_count else 0,
                    created_at=datetime.utcnow()
                )
                db.add(new_asset)
            
            # 4. Sync tags
            if tags:
                tag_list = [t.strip() for t in tags.split(',')]
                for tag in tag_list:
                    existing_tag = db.query(AssetTag).filter(
                        AssetTag.asset_id == (existing_asset.id if existing_asset else new_asset.id),
                        AssetTag.tag == tag
                    ).first()
                    
                    if not existing_tag:
                        new_tag = AssetTag(
                            asset_id=existing_asset.id if existing_asset else new_asset.id,
                            tag=tag
                        )
                        db.add(new_tag)
        
        db.commit()
        logger.info(f"Synced {len(rows)} assets from Google Sheets")
        
    except Exception as e:
        logger.error(f"Error syncing assets: {e}")
        db.rollback()
        raise
```

### 3. Semantic Search

```python
from sqlalchemy import or_, and_
from sqlalchemy.orm import Session

async def search_assets(
    db: Session,
    tags: List[str] = None,
    usage_role: str = None,
    campaign: str = None,
    asset_type: str = None,
    limit: int = 50,
    offset: int = 0
) -> List[Asset]:
    """
    Search assets by tags, usage role, campaign, or type
    """
    
    query = db.query(Asset)
    
    # Filter by asset type
    if asset_type:
        query = query.filter(Asset.type == asset_type)
    
    # Filter by usage role
    if usage_role:
        query = query.filter(Asset.usage_role == usage_role)
    
    # Filter by campaign
    if campaign:
        query = query.filter(Asset.campaign_or_scene.ilike(f"%{campaign}%"))
    
    # Filter by tags (OR condition - match any tag)
    if tags:
        tag_filters = []
        for tag in tags:
            tag_filters.append(
                Asset.id.in_(
                    db.query(AssetTag.asset_id).filter(
                        AssetTag.tag.ilike(f"%{tag}%")
                    )
                )
            )
        
        if tag_filters:
            query = query.filter(or_(*tag_filters))
    
    # Sort by usage count (most used first)
    query = query.order_by(Asset.usage_count.desc())
    
    # Paginate
    total = query.count()
    assets = query.limit(limit).offset(offset).all()
    
    return {
        'total': total,
        'limit': limit,
        'offset': offset,
        'assets': assets
    }
```

### 4. Asset Selection for Story Generation

```python
async def select_best_asset(
    db: Session,
    scene: Scene,
    asset_type: str,
    campaign: str = None
) -> Asset:
    """
    Intelligently select the best asset for a scene
    """
    
    # 1. Extract keywords from scene description
    keywords = extract_keywords(scene.visual_concept)
    
    # 2. Search for matching assets
    candidates = await search_assets(
        db,
        tags=keywords,
        usage_role='hero' if scene.scene_number == 1 else 'supporting',
        campaign=campaign,
        asset_type=asset_type,
        limit=10
    )
    
    if not candidates['assets']:
        # Fallback: search without tags
        candidates = await search_assets(
            db,
            usage_role='supporting',
            asset_type=asset_type,
            limit=10
        )
    
    # 3. Score candidates
    best_asset = None
    best_score = 0
    
    for asset in candidates['assets']:
        score = calculate_asset_score(asset, scene, keywords)
        
        if score > best_score:
            best_score = score
            best_asset = asset
    
    # 4. Track usage
    if best_asset:
        usage = AssetUsage(
            asset_id=best_asset.id,
            project_id=scene.project_id,
            story_id=scene.story_id,
            used_at=datetime.utcnow()
        )
        db.add(usage)
        
        # Update usage count
        best_asset.usage_count += 1
        best_asset.last_used = datetime.utcnow()
        
        db.commit()
    
    return best_asset

def calculate_asset_score(asset: Asset, scene: Scene, keywords: List[str]) -> float:
    """
    Calculate relevance score for asset
    
    Factors:
    - Tag matches (40%)
    - Usage role match (30%)
    - Recency of usage (20%)
    - Campaign match (10%)
    """
    
    score = 0.0
    
    # Tag matching
    asset_tags = {tag.tag.lower() for tag in asset.asset_tags}
    keyword_matches = sum(1 for kw in keywords if kw.lower() in asset_tags)
    tag_score = (keyword_matches / len(keywords)) if keywords else 0
    score += tag_score * 0.4
    
    # Usage role match
    if asset.usage_role == scene.usage_role:
        score += 0.3
    
    # Recency (prefer recently used assets)
    days_since_used = (datetime.utcnow() - asset.last_used).days
    recency_score = max(0, 1 - (days_since_used / 365))
    score += recency_score * 0.2
    
    # Campaign match
    if asset.campaign_or_scene and scene.campaign:
        if asset.campaign_or_scene.lower() == scene.campaign.lower():
            score += 0.1
    
    return score
```

### 5. Asset Usage Tracking

```python
async def track_asset_usage(
    asset_id: str,
    project_id: str,
    story_id: str,
    usage_type: str = 'story_generation'
):
    """Track how assets are used in stories"""
    
    usage = AssetUsage(
        asset_id=asset_id,
        project_id=project_id,
        story_id=story_id,
        used_at=datetime.utcnow()
    )
    
    db.add(usage)
    
    # Update asset metadata
    asset = db.query(Asset).filter(Asset.id == asset_id).first()
    if asset:
        asset.usage_count += 1
        asset.last_used = datetime.utcnow()
    
    db.commit()
    
    # Sync back to Google Sheets
    await update_google_sheets_usage(asset_id, asset.usage_count)

async def update_google_sheets_usage(asset_id: str, usage_count: int):
    """Update usage count in Google Sheets"""
    
    drive_manager = GoogleDriveManager()
    
    # Find row with this asset_id
    sheet_result = drive_manager.sheets_service.spreadsheets().values().get(
        spreadsheetId=drive_manager.sheet_id,
        range='SIRINX_Media_Asset_DB!A:A'
    ).execute()
    
    rows = sheet_result.get('values', [])
    
    for i, row in enumerate(rows):
        if row and row[0] == asset_id:
            # Update usage_count in column I (index 8)
            drive_manager.sheets_service.spreadsheets().values().update(
                spreadsheetId=drive_manager.sheet_id,
                range=f'SIRINX_Media_Asset_DB!I{i+1}',
                valueInputOption='RAW',
                body={'values': [[usage_count]]}
            ).execute()
            break
```

### 6. Asset Caching

```python
import redis
import json

class AssetCache:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
        self.ttl = 3600  # 1 hour
    
    async def get_cached_assets(self, cache_key: str) -> List[Asset]:
        """Get cached search results"""
        
        cached = self.redis.get(cache_key)
        if cached:
            return json.loads(cached)
        return None
    
    async def cache_assets(self, cache_key: str, assets: List[Asset]):
        """Cache search results"""
        
        self.redis.setex(
            cache_key,
            self.ttl,
            json.dumps([asset.to_dict() for asset in assets])
        )
    
    def generate_cache_key(
        self,
        tags: List[str] = None,
        usage_role: str = None,
        campaign: str = None,
        asset_type: str = None
    ) -> str:
        """Generate cache key from search parameters"""
        
        key_parts = [
            'assets',
            ','.join(sorted(tags)) if tags else 'notags',
            usage_role or 'norole',
            campaign or 'nocampaign',
            asset_type or 'notype'
        ]
        
        return ':'.join(key_parts)
    
    async def invalidate_cache(self, asset_id: str):
        """Invalidate cache when asset is updated"""
        
        # Delete all cache entries (simple approach)
        # In production, use more sophisticated invalidation
        pattern = 'assets:*'
        for key in self.redis.scan_iter(match=pattern):
            self.redis.delete(key)
```

---

## API Endpoints

### Search Assets

```http
GET /api/assets?tags=electricity,business&usage_role=hero&campaign=Q1-2026&limit=20
Authorization: Bearer {token}

Response:
{
  "total": 245,
  "limit": 20,
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
      "last_used": "2026-04-15T14:30:00Z"
    }
  ]
}
```

### Get Asset Details

```http
GET /api/assets/{asset_id}
Authorization: Bearer {token}

Response:
{
  "id": "uuid",
  "name": "SME-Electricity-Hero-1.jpg",
  "type": "image",
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

### Sync Google Drive

```http
POST /api/assets/sync
Authorization: Bearer {token}

Response:
{
  "sync_id": "uuid",
  "status": "completed",
  "total_assets": 1245,
  "synced_count": 1245,
  "new_assets": 45,
  "updated_assets": 120,
  "deleted_assets": 5
}
```

---

## Performance Optimization

### Indexing Strategy

```sql
-- Fast tag search
CREATE INDEX idx_asset_tags_tag ON asset_tags(tag);

-- Fast role search
CREATE INDEX idx_assets_usage_role ON assets(usage_role);

-- Fast campaign search
CREATE INDEX idx_assets_campaign ON assets(campaign_or_scene);

-- Composite index for common queries
CREATE INDEX idx_assets_role_campaign ON assets(usage_role, campaign_or_scene);
```

### Caching Strategy

- **Search Results**: Cache for 1 hour
- **Asset Metadata**: Cache for 24 hours
- **Google Drive Sync**: Cache for 6 hours
- **Preview URLs**: Cache for 7 days

---

**Document Version:** 1.0  
**Last Updated:** April 18, 2026  
**Status:** Ready for Implementation
