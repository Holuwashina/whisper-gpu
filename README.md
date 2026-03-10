# Whisper GPU Setup (Speaches TTS)

Local text-to-speech server using Speaches with GPU acceleration.

## GPU Compatibility

- **NVIDIA Quadro M1000M** (2GB VRAM) - Maxwell architecture
- May work via CUDA 12.x PTX compatibility
- CPU fallback supported if GPU fails

## Docker Setup

### Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) installed

### Start GPU Server
```bash
# Clone the repo
git clone https://github.com/Holuwashina/whisper-gpu.git
cd whisper-gpu

# Start with GPU support
$env:COMPOSE_FILE = "compose.yaml:compose.cuda.yaml"  # PowerShell
docker compose up --detach
```

### Verify Running
```bash
docker ps
# Should show: speaches container running on port 8000

curl http://localhost:8000/health
```

## Download TTS Model

```bash
$env:SPEACHES_BASE_URL = "http://localhost:8000"

# Download a TTS model
uvx speaches-cli model download speaches-ai/Kokoro-82M-v1.0-ONNX
```

## Test TTS (cURL)

```bash
$env:SPEACHES_BASE_URL = "http://localhost:8000"
$env:SPEECH_MODEL_ID = "speaches-ai/Kokoro-82M-v1.0-ONNX"
$env:VOICE_ID = "af_heart"

# Generate speech
curl "$env:SPEACHES_BASE_URL/v1/audio/speech" `
  -s -H "Content-Type: application/json" `
  --output audio.mp3 `
  --data-raw '{
    "input": "Hello World!",
    "model": "'$env:SPEECH_MODEL_ID'",
    "voice": "'$env:VOICE_ID'"
  }'
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
    },
).raise_for_status()

with Path("output.mp3").open("wb") as f:
    f.write(res.read())
```

## Common Commands

```bash
# View logs
docker compose logs -f speaches

# Stop container
docker compose down

# Restart container
docker compose restart

# Pull latest image
docker compose pull
```

## Troubleshooting

If GPU mode fails with CUDA errors, verify NVIDIA Container Toolkit is installed:

```bash
docker run --rm --gpus all nvidia/cuda:12.6.3-cudnn-runtime-ubuntu24.04 nvidia-sme
```
