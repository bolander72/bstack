---
name: kokoro-tts
description: "Generate speech audio from text using local TTS models (mlx-audio pocket-tts or Kokoro-ONNX). Use when user asks to generate a voice message, text-to-speech, speak something aloud, create an audio clip from text, or send a voice reply. Also use for iMessage voice bubble replies. Runs entirely locally — free, no API keys. NOT for speech-to-text or transcription (that's whisper/STT)."
---

# kokoro-tts — Local Text-to-Speech

Project: https://github.com/lucasnewman/kokoro-onnx (Kokoro-ONNX)
Alt: https://github.com/ml-explore/mlx-audio (mlx-audio pocket-tts)

Generate speech audio from text using local models on Apple Silicon. Zero cost, no API keys, no network required.

## Models

| Model | Library | Size | Notes |
|-------|---------|------|-------|
| pocket-tts-4bit | mlx-audio | ~200MB | No named voices, auto-detects language. Fastest on Apple Silicon. |
| kokoro-v1.0 | kokoro-onnx | ~82MB | Named voices (af_bella, am_puck, ef_dora, etc.), explicit language selection. |

Both produce high-quality speech at near-realtime speeds on M-series Macs.

## Prerequisites

### mlx-audio (recommended)
```bash
pip install mlx-audio soundfile numpy
```

### Kokoro-ONNX (alternative)
```bash
pip install kokoro-onnx soundfile numpy
# Models auto-download to ~/.cache/kokoro-onnx/ on first use
```

### For iMessage voice bubbles (CAF/Opus format)
macOS `afconvert` is required (pre-installed on all Macs).

## Quick Usage

### Generate WAV audio
```python
# mlx-audio
from mlx_audio.tts.utils import load_model
import soundfile as sf
import numpy as np

model = load_model("mlx-community/pocket-tts-4bit")
results = list(model.generate("Hello, this is a test."))
sf.write("output.wav", np.array(results[0].audio), results[0].sample_rate)
```

```python
# Kokoro-ONNX
from kokoro_onnx import Kokoro
import soundfile as sf

kokoro = Kokoro("~/.cache/kokoro-onnx/kokoro-v1.0.onnx",
                "~/.cache/kokoro-onnx/voices-v1.0.bin")
samples, sr = kokoro.create("Hello, this is a test.", voice="af_bella", speed=1.0)
sf.write("output.wav", samples, sr)
```

### Generate iMessage Voice Bubble (CAF/Opus)

iMessage voice messages require CAF container with Opus codec at 48kHz mono. Use macOS `afconvert`:

```bash
# Step 1: Generate WAV (using either model above)
# Step 2: Resample to 48kHz mono
afconvert input.wav -o resampled.wav -d LEI16 -c 1 -r 48000
# Step 3: Convert to CAF/Opus
afconvert resampled.wav -o voice_reply.caf -f caff -d opus -b 32000
```

The output `.caf` file plays as a native voice bubble in iMessage (with waveform, not as a file attachment).

### Prepend silence (recommended)
iMessage's audio player clips the first ~100ms. Prepend 150ms silence:
```python
silence = np.zeros(int(sample_rate * 0.15), dtype=samples.dtype)
samples = np.concatenate([silence, samples])
```

## Sending via BlueBubbles

After generating the .caf file, send as a voice message:
```
message action=upload-file channel=bluebubbles target=<recipient>
  filePath=/path/to/voice_reply.caf
  filename="Audio Message.caf"
  contentType="audio/x-caf"
```

Always include a text reply alongside voice messages for accessibility.

## Kokoro-ONNX Voice Reference

| Voice ID | Description | Language |
|----------|-------------|----------|
| af_bella | Female, warm | English |
| af_heart | Female, gentle | English |
| am_puck | Male, clear | English |
| am_adam | Male, deep | English |
| ef_dora | Female | Spanish |
| em_alex | Male | Spanish |

Full list: https://github.com/lucasnewman/kokoro-onnx#voices

## Tips

- **Speed control:** Kokoro-ONNX supports `speed=` param (0.5-2.0). mlx-audio pocket-tts does not.
- **Language:** pocket-tts auto-detects. Kokoro requires explicit `lang=` param ("en-us", "es", etc.).
- **Batch:** For long text, split into sentences and generate each separately for lower latency.
- **Streaming:** For real-time applications, generate sentence-by-sentence as LLM output streams in.
- **Quality:** Both models produce natural-sounding speech. Kokoro has more voice variety; pocket-tts is simpler.
