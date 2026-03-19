# Multimodal Apps (Vision, Audio, Images)

## Vision — Analyzing Images with Claude

Claude can analyze images passed as base64 or URL.

```python
import anthropic
import base64
from pathlib import Path

client = anthropic.Anthropic()

# Method 1: From file (base64)
def analyze_image_file(image_path: str, question: str) -> str:
    image_data = Path(image_path).read_bytes()
    b64_image = base64.standard_b64encode(image_data).decode("utf-8")

    # Detect media type
    ext = Path(image_path).suffix.lower()
    media_type_map = {
        ".jpg": "image/jpeg", ".jpeg": "image/jpeg",
        ".png": "image/png", ".gif": "image/gif",
        ".webp": "image/webp"
    }
    media_type = media_type_map.get(ext, "image/jpeg")

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": [
                {
                    "type": "image",
                    "source": {
                        "type": "base64",
                        "media_type": media_type,
                        "data": b64_image
                    }
                },
                {
                    "type": "text",
                    "text": question
                }
            ]
        }]
    )
    return response.content[0].text

# Method 2: From URL
def analyze_image_url(image_url: str, question: str) -> str:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": [
                {
                    "type": "image",
                    "source": {
                        "type": "url",
                        "url": image_url
                    }
                },
                {
                    "type": "text",
                    "text": question
                }
            ]
        }]
    )
    return response.content[0].text

# Usage
description = analyze_image_file("screenshot.png", "What is shown in this image?")
print(description)
```

---

## Common Vision Use Cases

### Screenshot/UI Analysis
```python
def analyze_ui(screenshot_path: str) -> dict:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": [
                {"type": "image", "source": {"type": "base64", "media_type": "image/png",
                    "data": encode_image(screenshot_path)}},
                {"type": "text", "text": """Analyze this UI screenshot and return JSON:
{
  "page_type": "login/dashboard/form/etc",
  "main_elements": ["list of key UI elements"],
  "issues": ["any UX/accessibility issues"],
  "description": "brief description"
}"""}
            ]
        }]
    )
    import json
    return json.loads(response.content[0].text)
```

### Document/Receipt Extraction
```python
def extract_receipt_data(image_path: str) -> dict:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        messages=[{
            "role": "user",
            "content": [
                {"type": "image", "source": {"type": "base64", "media_type": "image/jpeg",
                    "data": encode_image(image_path)}},
                {"type": "text", "text": """Extract receipt data as JSON:
{
  "merchant": "store name",
  "date": "YYYY-MM-DD",
  "items": [{"name": "...", "price": 0.00}],
  "subtotal": 0.00,
  "tax": 0.00,
  "total": 0.00
}"""}
            ]
        }]
    )
    import json
    return json.loads(response.content[0].text)
```

### Multi-Image Analysis
```python
def compare_images(image1_path: str, image2_path: str) -> str:
    content = [
        {"type": "image", "source": {"type": "base64", "media_type": "image/png",
            "data": encode_image(image1_path)}},
        {"type": "text", "text": "Image 1 above. Image 2 below:"},
        {"type": "image", "source": {"type": "base64", "media_type": "image/png",
            "data": encode_image(image2_path)}},
        {"type": "text", "text": "What are the key differences between these two images?"}
    ]

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        messages=[{"role": "user", "content": content}]
    )
    return response.content[0].text

def encode_image(path: str) -> str:
    return base64.standard_b64encode(Path(path).read_bytes()).decode("utf-8")
```

---

## Audio — Transcription with OpenAI Whisper

```python
from openai import OpenAI

client = OpenAI()

def transcribe_audio(audio_path: str) -> str:
    with open(audio_path, "rb") as f:
        transcript = client.audio.transcriptions.create(
            model="whisper-1",
            file=f,
            response_format="text"
        )
    return transcript

def transcribe_with_timestamps(audio_path: str) -> dict:
    with open(audio_path, "rb") as f:
        transcript = client.audio.transcriptions.create(
            model="whisper-1",
            file=f,
            response_format="verbose_json",
            timestamp_granularities=["segment"]
        )
    return transcript

# Transcribe then process with Claude
def meeting_summary(audio_path: str) -> str:
    transcript = transcribe_audio(audio_path)

    import anthropic
    claude = anthropic.Anthropic()
    response = claude.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": f"""Here is a meeting transcript. Provide:
1. Key decisions made
2. Action items with owners
3. Summary (3-5 sentences)

Transcript:
{transcript}"""
        }]
    )
    return response.content[0].text
```

---

## Image Generation with DALL-E

```python
from openai import OpenAI

client = OpenAI()

def generate_image(prompt: str, size: str = "1024x1024") -> str:
    response = client.images.generate(
        model="dall-e-3",
        prompt=prompt,
        size=size,         # "1024x1024", "1792x1024", "1024x1792"
        quality="standard", # "standard" or "hd"
        n=1
    )
    return response.data[0].url

def edit_image(image_path: str, mask_path: str, instruction: str) -> str:
    with open(image_path, "rb") as img, open(mask_path, "rb") as mask:
        response = client.images.edit(
            model="dall-e-2",
            image=img,
            mask=mask,
            prompt=instruction,
            n=1,
            size="1024x1024"
        )
    return response.data[0].url
```

---

## PDF Processing with Vision

For PDFs with complex layouts, convert pages to images then use vision:

```bash
pip install pypdf pillow pdf2image
```

```python
from pdf2image import convert_from_path
import io
import base64

def pdf_to_images(pdf_path: str) -> list[str]:
    """Convert PDF pages to base64 images."""
    images = convert_from_path(pdf_path, dpi=200)
    result = []
    for img in images:
        buf = io.BytesIO()
        img.save(buf, format="PNG")
        result.append(base64.standard_b64encode(buf.getvalue()).decode("utf-8"))
    return result

def analyze_pdf_visual(pdf_path: str, question: str) -> str:
    pages = pdf_to_images(pdf_path)

    content = []
    for i, page_b64 in enumerate(pages[:10]):  # limit to 10 pages
        content.append({"type": "text", "text": f"Page {i+1}:"})
        content.append({
            "type": "image",
            "source": {"type": "base64", "media_type": "image/png", "data": page_b64}
        })

    content.append({"type": "text", "text": question})

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        messages=[{"role": "user", "content": content}]
    )
    return response.content[0].text
```
