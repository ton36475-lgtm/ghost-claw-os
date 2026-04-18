# Ghost Claw OS - API Integration Guide

**Version:** 1.0  
**Status:** Ready for Implementation  
**Last Updated:** April 18, 2026

---

## 🔗 Overview

This guide explains how to integrate external APIs into Ghost Claw OS:
- **OpenAI** - LLM for story generation and content creation
- **Grok** - Advanced prompt optimization and reasoning
- **Google Drive** - Asset library and metadata storage
- **YouTube, TikTok, Instagram, LinkedIn** - Publishing platforms

---

## 1️⃣ OpenAI Integration

### **Setup**

```bash
# Install OpenAI SDK
pip install openai

# Set API key
export OPENAI_API_KEY="sk-..."
```

### **Configuration**

```python
# packages/api-fastapi/app/config.py

from openai import OpenAI

class OpenAIConfig:
    def __init__(self):
        self.client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
        self.model = "gpt-4-turbo"
        self.max_tokens = 2000
        self.temperature = 0.7
    
    async def generate_story_structure(self, topic: str, platforms: list) -> dict:
        """Generate PAS story structure using GPT-4"""
        prompt = f"""
        Create a compelling story structure for: {topic}
        
        Platforms: {', '.join(platforms)}
        
        Format as JSON with:
        - hook: Opening hook (1-2 sentences)
        - problem: Problem statement
        - agitate: Why it matters
        - solve: Solution/insight
        - cta: Call to action
        """
        
        response = await self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=self.max_tokens,
            temperature=self.temperature
        )
        
        return json.loads(response.choices[0].message.content)
    
    async def generate_voiceover(self, story: dict) -> str:
        """Generate voiceover script"""
        prompt = f"""
        Write a professional voiceover script for this story:
        {json.dumps(story, ensure_ascii=False)}
        
        Requirements:
        - Thai language
        - 6-8 seconds duration
        - Professional tone
        - Clear pronunciation
        """
        
        response = await self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=500
        )
        
        return response.choices[0].message.content
```

### **Usage in Workers**

```python
# packages/ml-workers/workers/story_generation_worker.py

class StoryGenerationWorker:
    def __init__(self):
        self.openai = OpenAIConfig()
    
    async def process_job(self, job_data: dict):
        # Generate story structures
        stories = []
        for i in range(8):
            structure = await self.openai.generate_story_structure(
                topic=job_data["topic"],
                platforms=job_data["platforms"]
            )
            
            # Generate voiceover
            voiceover = await self.openai.generate_voiceover(structure)
            
            stories.append({
                **structure,
                "voiceover_script": voiceover
            })
        
        return {"stories": stories}
```

### **Cost Optimization**

```python
# Batch requests for cost savings
async def batch_generate_stories(topics: list):
    """Generate multiple stories in one batch"""
    batch_requests = []
    
    for topic in topics:
        batch_requests.append({
            "custom_id": f"story-{topic}",
            "params": {
                "model": "gpt-4-turbo",
                "messages": [{"role": "user", "content": f"Generate story for: {topic}"}]
            }
        })
    
    # Submit batch
    response = await client.batches.create(requests=batch_requests)
    
    return response.id
```

---

## 2️⃣ Grok Integration

### **Setup**

```bash
# Install Grok SDK (via xAI)
pip install xai-sdk

# Set API key
export GROK_API_KEY="..."
```

### **Configuration**

```python
# packages/api-fastapi/app/config.py

from xai import Grok

class GrokConfig:
    def __init__(self):
        self.client = Grok(api_key=os.getenv("GROK_API_KEY"))
        self.model = "grok-1"
    
    async def optimize_prompt(self, prompt: str, context: dict) -> str:
        """Optimize prompt using Grok's reasoning"""
        optimization_prompt = f"""
        Analyze and optimize this prompt for better results:
        
        Original Prompt:
        {prompt}
        
        Context:
        {json.dumps(context, ensure_ascii=False)}
        
        Provide:
        1. Optimized prompt
        2. Reasoning for changes
        3. Expected improvement
        """
        
        response = await self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": optimization_prompt}]
        )
        
        return response.choices[0].message.content
    
    async def generate_content_flow(self, topic: str, platforms: list) -> dict:
        """Generate content flow using Grok's reasoning"""
        prompt = f"""
        Design an optimal content flow for: {topic}
        Platforms: {', '.join(platforms)}
        
        Consider:
        - Platform-specific best practices
        - Audience psychology
        - Engagement metrics
        - Content sequencing
        
        Return as structured JSON with:
        - flow_stages: Sequential content stages
        - timing: Optimal timing for each stage
        - metrics: Expected engagement metrics
        - optimization: Continuous improvement strategies
        """
        
        response = await self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": prompt}]
        )
        
        return json.loads(response.choices[0].message.content)
```

### **Usage in Prompt Lab**

```python
# packages/api-fastapi/app/routers/prompts.py

@router.post("/api/prompts/optimize")
async def optimize_prompt(prompt_data: dict):
    """Optimize prompt using Grok"""
    grok = GrokConfig()
    
    optimized = await grok.optimize_prompt(
        prompt=prompt_data["prompt"],
        context=prompt_data.get("context", {})
    )
    
    return {
        "original": prompt_data["prompt"],
        "optimized": optimized
    }

@router.post("/api/content-flow/generate")
async def generate_content_flow(topic: str, platforms: list):
    """Generate content flow using Grok"""
    grok = GrokConfig()
    
    flow = await grok.generate_content_flow(topic, platforms)
    
    return flow
```

---

## 3️⃣ Google Drive Integration

### **Setup**

```bash
# Install Google client library
pip install google-auth-oauthlib google-auth-httplib2 google-api-python-client

# Create service account
# 1. Go to Google Cloud Console
# 2. Create service account
# 3. Download JSON key
# 4. Save to /path/to/service-account.json
```

### **Configuration**

```python
# packages/api-fastapi/app/config.py

from google.oauth2 import service_account
from googleapiclient.discovery import build
from googleapiclient.http import MediaFileUpload

class GoogleDriveConfig:
    def __init__(self):
        self.credentials = service_account.Credentials.from_service_account_file(
            os.getenv("GOOGLE_SERVICE_ACCOUNT_JSON"),
            scopes=['https://www.googleapis.com/auth/drive']
        )
        self.drive_service = build('drive', 'v3', credentials=self.credentials)
        self.sheets_service = build('sheets', 'v4', credentials=self.credentials)
    
    async def list_assets(self, folder_id: str) -> list:
        """List all assets in a folder"""
        results = self.drive_service.files().list(
            q=f"'{folder_id}' in parents and trashed=false",
            spaces='drive',
            fields='files(id, name, mimeType, webViewLink, createdTime)',
            pageSize=100
        ).execute()
        
        return results.get('files', [])
    
    async def get_file_metadata(self, file_id: str) -> dict:
        """Get detailed metadata for a file"""
        file = self.drive_service.files().get(
            fileId=file_id,
            fields='*'
        ).execute()
        
        return file
    
    async def upload_asset(self, file_path: str, folder_id: str, metadata: dict) -> str:
        """Upload asset to Google Drive"""
        file_metadata = {
            'name': os.path.basename(file_path),
            'parents': [folder_id],
            'properties': metadata
        }
        
        media = MediaFileUpload(file_path)
        
        file = self.drive_service.files().create(
            body=file_metadata,
            media_body=media,
            fields='id'
        ).execute()
        
        return file.get('id')
    
    async def sync_metadata_to_sheets(self, sheet_id: str, assets: list):
        """Sync asset metadata to Google Sheets"""
        values = [
            ['Asset Name', 'Type', 'Tags', 'Usage Role', 'Campaign', 'Drive URL']
        ]
        
        for asset in assets:
            values.append([
                asset.get('name'),
                asset.get('type'),
                ', '.join(asset.get('tags', [])),
                asset.get('usage_role'),
                asset.get('campaign'),
                asset.get('drive_url')
            ])
        
        body = {'values': values}
        
        self.sheets_service.spreadsheets().values().update(
            spreadsheetId=sheet_id,
            range='Sheet1!A1',
            valueInputOption='RAW',
            body=body
        ).execute()
```

### **Asset Memory Implementation**

```python
# packages/api-fastapi/app/routers/assets.py

@router.post("/api/assets/sync")
async def sync_google_drive():
    """Sync assets from Google Drive"""
    drive = GoogleDriveConfig()
    
    # List all assets
    folder_id = os.getenv("GOOGLE_DRIVE_FOLDER_ID")
    assets = await drive.list_assets(folder_id)
    
    # Enrich with metadata
    enriched_assets = []
    for asset in assets:
        metadata = await drive.get_file_metadata(asset['id'])
        enriched_assets.append({
            **asset,
            'metadata': metadata.get('properties', {})
        })
    
    # Sync to database
    for asset in enriched_assets:
        db.assets.insert_one({
            'drive_id': asset['id'],
            'name': asset['name'],
            'type': asset.get('mimeType'),
            'url': asset.get('webViewLink'),
            'metadata': asset.get('metadata', {}),
            'synced_at': datetime.utcnow()
        })
    
    # Sync to Google Sheets
    sheet_id = os.getenv("GOOGLE_SHEETS_ID")
    await drive.sync_metadata_to_sheets(sheet_id, enriched_assets)
    
    return {"synced": len(enriched_assets)}

@router.get("/api/assets/search")
async def search_assets(tags: list = None, usage_role: str = None, campaign: str = None):
    """Search assets by metadata"""
    query = {}
    
    if tags:
        query['metadata.tags'] = {'$in': tags}
    if usage_role:
        query['metadata.usage_role'] = usage_role
    if campaign:
        query['metadata.campaign'] = campaign
    
    assets = db.assets.find(query)
    
    return list(assets)
```

---

## 4️⃣ Platform Publishing APIs

### **YouTube Integration**

```python
# packages/api-fastapi/app/config.py

from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

class YouTubeConfig:
    def __init__(self):
        self.youtube = build('youtube', 'v3', credentials=self.get_credentials())
    
    async def upload_video(self, file_path: str, metadata: dict) -> str:
        """Upload video to YouTube"""
        body = {
            'snippet': {
                'title': metadata['title'],
                'description': metadata['description'],
                'tags': metadata.get('tags', []),
                'categoryId': '22'  # People & Blogs
            },
            'status': {
                'privacyStatus': 'public'
            }
        }
        
        media = MediaFileUpload(file_path, chunksize=1024*1024, resumable=True)
        
        request = self.youtube.videos().insert(
            part='snippet,status',
            body=body,
            media_body=media
        )
        
        response = request.execute()
        return response['id']
    
    async def add_subtitles(self, video_id: str, subtitle_file: str, language: str):
        """Add subtitles to video"""
        body = {
            'snippet': {
                'videoId': video_id,
                'language': language,
                'name': f'Subtitles - {language}'
            }
        }
        
        media = MediaFileUpload(subtitle_file)
        
        request = self.youtube.captions().insert(
            part='snippet',
            body=body,
            media_body=media
        )
        
        request.execute()
```

### **TikTok Integration**

```python
# packages/api-fastapi/app/config.py

import aiohttp

class TikTokConfig:
    def __init__(self):
        self.api_url = "https://open.tiktok.com/v1"
        self.access_token = os.getenv("TIKTOK_ACCESS_TOKEN")
    
    async def upload_video(self, file_path: str, metadata: dict) -> str:
        """Upload video to TikTok"""
        async with aiohttp.ClientSession() as session:
            # Initialize upload
            init_response = await session.post(
                f"{self.api_url}/video/upload/init",
                headers={"Authorization": f"Bearer {self.access_token}"},
                json={
                    "source_info": {
                        "source": "FILE_UPLOAD",
                        "video_size": os.path.getsize(file_path)
                    }
                }
            )
            
            upload_token = init_response.json()['data']['upload_token']
            
            # Upload video file
            with open(file_path, 'rb') as f:
                await session.post(
                    f"{self.api_url}/video/upload/parts",
                    headers={"Authorization": f"Bearer {self.access_token}"},
                    params={"upload_token": upload_token, "part_number": 1},
                    data=f
                )
            
            # Complete upload
            complete_response = await session.post(
                f"{self.api_url}/video/upload/complete",
                headers={"Authorization": f"Bearer {self.access_token}"},
                json={
                    "upload_token": upload_token,
                    "post_info": {
                        "title": metadata['title'],
                        "description": metadata['description'],
                        "privacy_level": "PUBLIC_TO_ANYONE"
                    }
                }
            )
            
            return complete_response.json()['data']['video_id']
```

### **Instagram Integration**

```python
# packages/api-fastapi/app/config.py

class InstagramConfig:
    def __init__(self):
        self.graph_url = "https://graph.instagram.com"
        self.access_token = os.getenv("INSTAGRAM_ACCESS_TOKEN")
        self.business_account_id = os.getenv("INSTAGRAM_BUSINESS_ACCOUNT_ID")
    
    async def upload_video(self, file_path: str, metadata: dict) -> str:
        """Upload video to Instagram"""
        async with aiohttp.ClientSession() as session:
            # Create media container
            create_response = await session.post(
                f"{self.graph_url}/{self.business_account_id}/media",
                params={"access_token": self.access_token},
                json={
                    "media_type": "VIDEO",
                    "video_url": metadata['video_url'],
                    "caption": metadata['caption']
                }
            )
            
            media_id = create_response.json()['id']
            
            # Publish media
            publish_response = await session.post(
                f"{self.graph_url}/{self.business_account_id}/media_publish",
                params={"access_token": self.access_token},
                json={"creation_id": media_id}
            )
            
            return publish_response.json()['id']
```

---

## 5️⃣ Error Handling & Retry Logic

```python
# packages/api-fastapi/app/utils/retry.py

import asyncio
from functools import wraps

def retry_with_backoff(max_retries=3, backoff_factor=2):
    """Decorator for retrying API calls with exponential backoff"""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    
                    wait_time = backoff_factor ** attempt
                    logger.warning(f"Attempt {attempt + 1} failed, retrying in {wait_time}s: {str(e)}")
                    await asyncio.sleep(wait_time)
        
        return wrapper
    return decorator

# Usage
@retry_with_backoff(max_retries=3)
async def call_openai_api(prompt: str):
    return await openai_client.chat.completions.create(...)
```

---

## 6️⃣ Testing APIs

```python
# tests/test_api_integration.py

import pytest

@pytest.mark.asyncio
async def test_openai_story_generation():
    """Test OpenAI story generation"""
    config = OpenAIConfig()
    
    result = await config.generate_story_structure(
        topic="Test Topic",
        platforms=["youtube"]
    )
    
    assert 'hook' in result
    assert 'problem' in result
    assert 'solve' in result

@pytest.mark.asyncio
async def test_google_drive_sync():
    """Test Google Drive asset sync"""
    config = GoogleDriveConfig()
    
    assets = await config.list_assets(os.getenv("GOOGLE_DRIVE_FOLDER_ID"))
    
    assert isinstance(assets, list)
    assert len(assets) > 0

@pytest.mark.asyncio
async def test_youtube_upload():
    """Test YouTube upload"""
    config = YouTubeConfig()
    
    video_id = await config.upload_video(
        file_path="/tmp/test_video.mp4",
        metadata={
            "title": "Test Video",
            "description": "Test Description",
            "tags": ["test"]
        }
    )
    
    assert video_id is not None
```

---

## ✅ Checklist

- [ ] OpenAI API key configured
- [ ] Grok API key configured
- [ ] Google Drive service account setup
- [ ] Google Sheets ID configured
- [ ] YouTube API enabled
- [ ] TikTok API access granted
- [ ] Instagram Business API configured
- [ ] LinkedIn API configured
- [ ] All APIs tested
- [ ] Error handling implemented
- [ ] Rate limiting configured
- [ ] Monitoring setup

---

**Status:** Ready for Implementation  
**Version:** 1.0  
**Last Updated:** April 18, 2026
