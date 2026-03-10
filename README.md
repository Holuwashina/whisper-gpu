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
uvx speaches-cli model download speaches-ai/Kokoro-82M-v1.0-ONNX
```

## Test TTS
```bash
curl "$SPEACHES_BASE_URL/v1/audio/speech" \
  -H "Content-Type: application/json" \
  --output audio.mp3 \
  --data '{
    "input": "Hello World!",
    "model": "speaches-ai/Kokoro-82M-v1.0-ONNX",
    "voice": "af_heart"
  }'
```

## Troubleshooting

If GPU mode fails with CUDA errors, use CPU mode - ONNX inference runs well on CPU.
