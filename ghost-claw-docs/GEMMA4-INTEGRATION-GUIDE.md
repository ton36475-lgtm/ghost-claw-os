# Ghost Claw OS - Gemma 4 Integration Guide

**Version:** 1.0  
**Status:** Production-Ready  
**Last Updated:** April 18, 2026

---

## 🧠 Overview

**Gemma 4** (Google's Edge Generative AI) is now the core AI brain powering Ghost Claw OS. It handles all intelligent operations across the platform with support for edge devices (mobile, IoT) and local deployment.

### **Key Benefits**

✅ **Local Execution** - No cloud dependency, full privacy  
✅ **Edge Support** - Runs on mobile and IoT devices  
✅ **Cost Effective** - No API costs, one-time model download  
✅ **Fast Inference** - Optimized for real-time operations  
✅ **Multi-Platform** - CPU, GPU, and specialized hardware support  
✅ **Scalable** - From 2B to 27B parameter models  

---

## 🚀 Installation & Setup

### **Step 1: Install Dependencies**

```bash
# Install Python dependencies
pip install transformers torch accelerate bitsandbytes

# For GPU support (CUDA)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

# For Apple Silicon (Metal)
pip install torch torchvision torchaudio
```

### **Step 2: Download Gemma 4 Model**

```bash
# Accept license at https://huggingface.co/google/gemma-7b
# Then login to Hugging Face
huggingface-cli login

# Download model (choose one)
# For mobile/edge (2B model - ~5GB)
huggingface-cli download google/gemma-2b --local-dir ./models/gemma-2b

# For desktop (7B model - ~15GB)
huggingface-cli download google/gemma-7b --local-dir ./models/gemma-7b

# For high-performance (9B model - ~20GB)
huggingface-cli download google/gemma-9b --local-dir ./models/gemma-9b
```

### **Step 3: Configure Environment**

```bash
# Create .env file
cat > .env << EOF
# Gemma 4 Configuration
GEMMA_MODEL_SIZE=gemma-7b
GEMMA_EXECUTION_MODE=local_cpu
GEMMA_DEVICE=cpu
GEMMA_QUANTIZATION=int8
GEMMA_VERBOSE=true

# Model paths
GEMMA_MODEL_PATH=/models/gemma-7b
GEMMA_CACHE_DIR=/tmp/gemma_cache

# Performance
GEMMA_MAX_TOKENS=2048
GEMMA_TEMPERATURE=0.7
GEMMA_TOP_P=0.9
GEMMA_TOP_K=40
EOF
```

### **Step 4: Initialize Gemma 4**

```python
import asyncio
from config.gemma4_config import get_gemma4_brain

async def main():
    # Initialize Gemma 4 brain
    gemma4 = await get_gemma4_brain()
    
    # Test generation
    response = await gemma4.generate("Hello, what is AI?")
    print(response)

asyncio.run(main())
```

---

## 🎯 Configuration Presets

### **Mobile/Edge Devices**

```python
from config.gemma4_config import get_preset_config

# 2B model optimized for mobile
config = get_preset_config("mobile")
# - Model: gemma-2b
# - Memory: 2GB
# - Threads: 2
# - Quantization: int8
```

### **Desktop (CPU)**

```python
# 7B model for desktop CPU
config = get_preset_config("desktop")
# - Model: gemma-7b
# - Memory: 8GB
# - Threads: 8
# - Quantization: int8
```

### **Desktop (GPU)**

```python
# 9B model for GPU acceleration
config = get_preset_config("gpu")
# - Model: gemma-9b
# - Memory: 16GB
# - Device: CUDA/Metal
# - Quantization: fp16
```

### **Enterprise**

```python
# 27B model for high-performance
config = get_preset_config("enterprise")
# - Model: gemma-27b
# - Memory: 32GB
# - Device: Multi-GPU
# - Quantization: fp16
```

---

## 📊 Gemma 4 in Ghost Claw OS Modules

### **1. Story Generation Worker**

```python
from workers.gemma4_story_generation_worker import get_story_generation_worker

worker = await get_story_generation_worker()

# Generate 8 stories from topic
result = await worker.process_job({
    "topic": "ค่าไฟธุรกิจ SME ที่คาดเดาไม่ได้",
    "platforms": ["youtube", "tiktok", "instagram"],
    "style": "educational",
    "target_audience": "SME business owners",
    "cta": "Subscribe for more tips"
})

# Result contains 8 complete stories with:
# - Story structure (PAS framework)
# - Voiceover scripts
# - Visual concepts
# - On-screen text
# - Image prompts
# - Video prompts
# - Platform optimizations
```

### **2. Video Processing Worker**

```python
# Analyze video transcripts
analysis = await gemma4.analyze_video_content(
    transcript="...",
    metadata={"duration": 120, "language": "th"}
)

# Returns:
# - Key themes
# - Emotional moments
# - Engagement opportunities
# - Recommended cuts
```

### **3. Review Gate Worker**

```python
# Check compliance
compliance = await gemma4.review_compliance(
    content="Story content...",
    guidelines={
        "brand": "SIRINX",
        "tone": "professional",
        "platforms": ["youtube", "tiktok"]
    }
)

# Returns:
# - Compliance score (0-100)
# - Issues found
# - Recommendations
# - Auto-approval decision
```

### **4. Prompt Lab**

```python
# Optimize prompts
optimized = await gemma4.optimize_prompt(
    prompt="Original prompt...",
    context={"task": "story_generation", "platform": "youtube"}
)

# Returns improved prompt for better results
```

### **5. Asset Library**

```python
# Analyze assets
analysis = await gemma4.generate(
    prompt="Analyze this asset for use in story about electricity costs"
)

# Returns:
# - Asset relevance
# - Best use cases
# - Recommended combinations
# - Metadata suggestions
```

---

## ⚙️ API Reference

### **Core Methods**

#### **generate()**
```python
response = await gemma4.generate(
    prompt="Your prompt here",
    max_tokens=2048,
    temperature=0.7,
    top_p=0.9,
    top_k=40,
    system_prompt="Optional system message"
)
```

#### **generate_story_structure()**
```python
story = await gemma4.generate_story_structure(
    topic="Topic",
    platforms=["youtube", "tiktok"]
)
# Returns: {"hook": "...", "problem": "...", "solve": "..."}
```

#### **generate_voiceover()**
```python
script = await gemma4.generate_voiceover(
    story={"hook": "...", "problem": "..."},
    language="th"
)
# Returns: Voiceover script
```

#### **analyze_video_content()**
```python
analysis = await gemma4.analyze_video_content(
    transcript="Video transcript",
    metadata={"duration": 120}
)
# Returns: Analysis with themes, moments, opportunities
```

#### **review_compliance()**
```python
result = await gemma4.review_compliance(
    content="Content to review",
    guidelines={"brand": "SIRINX"}
)
# Returns: Compliance score, issues, recommendations
```

#### **optimize_prompt()**
```python
optimized = await gemma4.optimize_prompt(
    prompt="Original prompt",
    context={"task": "story_generation"}
)
# Returns: Improved prompt
```

#### **batch_generate()**
```python
responses = await gemma4.batch_generate(
    prompts=["prompt1", "prompt2", "prompt3"]
)
# Returns: List of responses
```

---

## 🔧 Performance Optimization

### **Memory Management**

```python
# Enable caching for repeated prompts
config.enable_cache = True
config.cache_dir = "/tmp/gemma_cache"

# Clear cache when needed
gemma4.clear_cache()

# Monitor cache size
stats = gemma4.get_stats()
print(f"Cache size: {stats['cache_size']} entries")
```

### **Inference Optimization**

```python
# Reduce max tokens for faster inference
await gemma4.generate(prompt, max_tokens=500)

# Lower temperature for deterministic results
await gemma4.generate(prompt, temperature=0.3)

# Use quantization for faster inference
config.quantization = "int8"  # Faster than fp16
```

### **Batch Processing**

```python
# Process multiple requests efficiently
prompts = [f"Generate story {i}" for i in range(8)]
results = await gemma4.batch_generate(prompts)

# Reduces overhead compared to individual calls
```

---

## 📱 Edge Device Deployment

### **Mobile (iOS/Android)**

```python
# Use Gemma 2B model
config = get_preset_config("mobile")

# Deploy with React Native
# Use TensorFlow Lite or ONNX Runtime
```

### **Raspberry Pi / IoT**

```bash
# Install on Raspberry Pi
pip install transformers torch

# Use Gemma 2B with quantization
export GEMMA_MODEL_SIZE=gemma-2b
export GEMMA_QUANTIZATION=int8
```

### **Offline Support**

```python
# All models run completely offline
# No internet required after model download
# Perfect for edge devices without connectivity
```

---

## 🧪 Testing & Validation

### **Unit Tests**

```python
import pytest

@pytest.mark.asyncio
async def test_story_generation():
    gemma4 = await get_gemma4_brain()
    
    story = await gemma4.generate_story_structure(
        topic="Test topic",
        platforms=["youtube"]
    )
    
    assert "hook" in story
    assert "problem" in story
    assert "solve" in story

@pytest.mark.asyncio
async def test_voiceover_generation():
    gemma4 = await get_gemma4_brain()
    
    script = await gemma4.generate_voiceover(
        story={"hook": "Test", "problem": "Test"},
        language="th"
    )
    
    assert len(script) > 0
```

### **Performance Tests**

```python
import time

async def benchmark_generation():
    gemma4 = await get_gemma4_brain()
    
    start = time.time()
    response = await gemma4.generate("Test prompt")
    elapsed = time.time() - start
    
    print(f"Generation time: {elapsed:.2f}s")
    print(f"Tokens per second: {len(response.split()) / elapsed:.2f}")
```

---

## 🚨 Troubleshooting

### **Issue: Out of Memory**

```python
# Solution 1: Use smaller model
config = get_preset_config("mobile")  # 2B instead of 7B

# Solution 2: Enable quantization
config.quantization = "int8"

# Solution 3: Reduce batch size
config.max_batch_size = 1
```

### **Issue: Slow Inference**

```python
# Solution 1: Use GPU
config.device = "cuda"

# Solution 2: Reduce max tokens
await gemma4.generate(prompt, max_tokens=500)

# Solution 3: Enable caching
config.enable_cache = True
```

### **Issue: Model Not Found**

```bash
# Download model again
huggingface-cli download google/gemma-7b --local-dir ./models/gemma-7b

# Verify model path in .env
export GEMMA_MODEL_PATH=/path/to/models/gemma-7b
```

---

## 📊 Monitoring & Metrics

### **Get Statistics**

```python
stats = gemma4.get_stats()
print(stats)
# Output:
# {
#   "initialized": True,
#   "model_size": "gemma-7b",
#   "execution_mode": "local_cpu",
#   "cache_size": 42,
#   "device": "cpu",
#   "quantization": "int8"
# }
```

### **Logging**

```python
import logging

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)

# All Gemma 4 operations will be logged
```

---

## 🔐 Security & Privacy

✅ **Local Execution** - No data sent to cloud  
✅ **No API Keys** - Complete privacy  
✅ **Offline Support** - Works without internet  
✅ **Model Quantization** - Reduces model size  
✅ **Input Validation** - All inputs sanitized  

---

## 📈 Performance Benchmarks

| Model | Device | Memory | Speed | Quality |
|-------|--------|--------|-------|---------|
| Gemma 2B | Mobile | 2GB | ⚡⚡⚡ | ⭐⭐⭐ |
| Gemma 7B | CPU | 8GB | ⚡⚡ | ⭐⭐⭐⭐ |
| Gemma 9B | GPU | 16GB | ⚡⚡⚡⚡ | ⭐⭐⭐⭐⭐ |
| Gemma 27B | Multi-GPU | 32GB | ⚡⚡⚡⚡⚡ | ⭐⭐⭐⭐⭐ |

---

## 🎓 Best Practices

1. **Use Appropriate Model Size**
   - Mobile: Gemma 2B
   - Desktop: Gemma 7B
   - GPU: Gemma 9B
   - Enterprise: Gemma 27B

2. **Enable Caching**
   - Reduces redundant inference
   - Improves response time
   - Saves computational resources

3. **Batch Processing**
   - Process multiple requests together
   - More efficient than individual calls
   - Better resource utilization

4. **Monitor Performance**
   - Track inference time
   - Monitor memory usage
   - Optimize based on metrics

5. **Regular Updates**
   - Keep Transformers library updated
   - Download latest model versions
   - Apply security patches

---

## 📚 Additional Resources

- [Gemma Model Card](https://huggingface.co/google/gemma-7b)
- [Transformers Documentation](https://huggingface.co/docs/transformers/)
- [PyTorch Documentation](https://pytorch.org/docs/)
- [Ghost Claw OS Architecture](./03-SYSTEM-ARCHITECTURE.md)

---

**Status:** ✅ Production-Ready  
**Version:** 1.0.0  
**Last Updated:** April 18, 2026  
**Maintained By:** Ghost Claw Studio
