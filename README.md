# Whisper GPU Setup (Speaches TTS)

Local text-to-speech server using Speaches with GPU acceleration.

## GPU Compatibility

- **NVIDIA Quadro M1000M** (2GB VRAM) - Maxwell architecture
- May work via CUDA 12.x PTX compatibility
- CPU fallback supported

## Quick Start

### GPU Mode
```bash
export COMPOSE_FILE=compose.yaml:compose.cuda.yaml
docker compose up --detach
```

### CPU Mode
```bash
export COMPOSE_FILE=compose.yaml:compose.cpu.yaml
docker compose up --detach
```

## Download TTS Model

```bash
export SPEACHES_BASE_URL="http://localhost:8000"

# List available TTS models
uvx speaches-cli registry ls --task text-to-speech | jq '.data | [].id'

# Download a TTS model
uvx speaches-cli model download speaches-ai/Kokoro-82M-v1.0-ONNX

# Verify model is installed
uvx speaches-cli model ls --task text-to-speech | jq '.data | map(select(.id == "speaches-ai/Kokoro-82M-v1.0-ONNX"))'
```

## Test TTS (cURL)

```bash
export SPEACHES_BASE_URL="http://localhost:8000"
export SPEECH_MODEL_ID="speaches-ai/Kokoro-82M-v1.0-ONNX"
export VOICE_ID="af_heart"

# Generate speech (MP3)
curl "$SPEACHES_BASE_URL/v1/audio/speech" \
  -s -H "Content-Type: application/json" \
  --output audio.mp3 \
  --data-raw "{
    \"input\": \"Hello World!\",
    \"model\": \"$SPEECH_MODEL_ID\",
    \"voice\": \"$VOICE_ID\"
  }"

# Generate speech (WAV)
curl "$SPEACHES_BASE_URL/v1/audio/speech" \
  -s -H "Content-Type: application/json" \
  --output audio.wav \
  --data-raw "{
    \"input\": \"Hello World!\",
    \"model\": \"$SPEECH_MODEL_ID\",
    \"voice\": \"$VOICE_ID\",
    \"response_format\": \"wav\"
  }"

# Generate with speed
curl "$SPEACHES_BASE_URL/v1/audio/speech" \
  -s -H "Content-Type: application/json" \
  --output audio.mp3 \
  --data-raw "{
    \"input\": \"Hello World!\",
    \"model\": \"$SPEECH_MODEL_ID\",
    \"voice\": \"$VOICE_ID\",
    \"speed\": 2.0
  }"
```

## Test TTS (Python)

```python
from pathlib import Path
import httpx

client = httpx.Client(base_url="http://localhost:8000/")
model_id = "speaches-ai/Kokoro-82M-v1.0-ONNX"
voice_id = "af_heart"

res = client.post(
    "v1/audio/speech",
    json={
        "model": model_id,
        "voice": voice_id,
        "input": "Hello, world!",
        "response_format": "mp3",
        "speed": 1,
    },
).raise_for_status()

with Path("output.mp3").open("wb") as f:
    f.write(res.read())
```

## Test TTS (OpenAI SDK)

```python
from pathlib import Path
from openai import OpenAI

openai = OpenAI(base_url="http://localhost:8000/v1", api_key="cant-be-empty")
model_id = "speaches-ai/Kokoro-82M-v1.0-ONNX"
voice_id = "af_heart"

res = openai.audio.speech.create(
    model=model_id,
    voice=voice_id,
    input="Hello, world!",
    response_format="mp3",
    speed=1,
)

with Path("output.mp3").open("wb") as f:
    f.write(res.response.read())
```

## Configuration

Environment variables can be set in a `.env` file:

```env
LOG_LEVEL=debug
PORT=8000
ENABLE_UI=true
```

## Troubleshooting

If GPU mode fails with CUDA errors, use CPU mode - ONNX inference runs well on CPU.

Check container logs:
```bash
docker compose logs -f speaches
```
