# Whisper GPU Setup (Speaches)

Local STT (Speech-to-Text) and TTS (Text-to-Speech) server using Speaches with GPU acceleration.

## GPU Compatibility

- **NVIDIA Quadro M1000M** (2GB VRAM) - Maxwell architecture
- May work via CUDA 12.x PTX compatibility

## Docker Setup

### Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) installed

### Start GPU Server
```bash
git clone https://github.com/Holuwashina/whisper-gpu.git
cd whisper-gpu

$env:COMPOSE_FILE = "compose.yaml:compose.cuda.yaml"
docker compose up --detach
```

### Verify Running
```bash
docker ps
curl http://localhost:8000/health
```

## Download Models

### Whisper (STT - Speech-to-Text)
```bash
$env:SPEACHES_BASE_URL = "http://localhost:8000"

# Download Whisper model
uvx speaches-cli model download Systran/faster-distil-whisper-small.en
```

### Kokoro (TTS - Text-to-Speech)
```bash
uvx speaches-cli model download speaches-ai/Kokoro-82M-v1.0-ONNX
```

## Test Whisper (STT)

```bash
$env:SPEACHES_BASE_URL = "http://localhost:8000"
$env:TRANSCRIPTION_MODEL_ID = "Systran/faster-distil-whisper-small.en"

curl -s "$env:SPEACHES_BASE_URL/v1/audio/transcriptions" `
  -F "file=@audio.wav" `
  -F "model=$env:TRANSCRIPTION_MODEL_ID"
```

## Test Kokoro (TTS)

```bash
$env:SPEACHES_BASE_URL = "http://localhost:8000"
$env:TTS_MODEL_ID = "speaches-ai/Kokoro-82M-v1.0-ONNX"
$env:VOICE_ID = "af_heart"

curl "$env:SPEACHES_BASE_URL/v1/audio/speech" `
  -s -H "Content-Type: application/json" `
  --output audio.mp3 `
  --data-raw '{
    "input": "Hello World!",
    "model": "'$env:TTS_MODEL_ID'",
    "voice": "'$env:VOICE_ID'"
  }'
```

## Common Commands

```bash
docker compose logs -f speaches
docker compose down
docker compose restart
docker compose pull
```

## Troubleshooting

Verify NVIDIA Container Toolkit:
```bash
docker run --rm --gpus all nvidia/cuda:12.6.3-cudnn-runtime-ubuntu24.04 nvidia-sme
```
