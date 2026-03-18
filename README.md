# Meeting Minutes Generator

A local, offline tool that takes a recorded meeting in MP3 format and spits out structured meeting minutes. No cloud services, no API keys — everything runs on your machine.

---

## What it does

You point it at an MP3 file, tell it how many people were in the meeting, and it handles the rest:

1. Converts the audio to WAV and normalises the volume
2. Runs speaker diarization to figure out who spoke when
3. Transcribes each speaker's segments using Whisper
4. Feeds the transcript through BART to extract discussion points, decisions, action items, questions, and concerns
5. Writes everything into a formatted `meeting_minutes.txt` file

There's a simple PyQt5 GUI so you don't have to touch the terminal, but you can also run the pipeline directly from the command line if you prefer.

---

## Requirements

- Python 3.9+
- Windows (the batch launcher is `.bat`, though the Python code itself is cross-platform)
- The models downloaded locally (see below)

**Python dependencies:**

```
torch
faster-whisper
ctranslate2
pyannote.audio
transformers
pydub
scipy
numpy
tqdm
PyQt5
tensorboardx
```

Install them into a virtual environment:

```bash
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

You'll also need `ffmpeg` and `ffprobe` binaries placed inside a `bin/` folder in the project root. The code looks for them there specifically.

---

## Models

Three models need to be present locally before you run anything. The expected directory layout under `models/` is:

```
models/
├── pyannote/
│   └── models--pyannote--speaker-diarization-3.1/
│       └── snapshots/
│           └── 84fd25912480287da0247647c3d2b4853cb3ee5d/
│               └── config.yaml  (+ other model files)
│
├── small/
│   └── models--Systran--faster-whisper-small/
│       └── snapshots/
│           └── 536b0662742c02347bc0e980a01041f333bce120/
│               └── model.bin  (+ other model files)
│
└── bart-samsum/
    └── models--facebook--bart-large-cnn/
        └── snapshots/
            └── 37f520fa929c961707657b28798b30c003dd100b/
                └── (model files)
```

You can download these from Hugging Face. Once they're in place, the tool runs fully offline.

> If you're unsure whether a downloaded `.whl` file has GPU support baked in, `download_models.py` can inspect it and tell you. Run it as `python download_models.py path_to_file.whl`.

---

## Running it

**With the GUI (recommended):**

```bash
run.bat
```

This activates the virtual environment and opens the interface. From there:
- Browse to your MP3 file
- Set the number of speakers
- Hit **Generate Minutes**

Progress is shown in a progress bar with stage labels (MP3 Conversion → Diarization → Transcription → Minute Generation). The finished minutes appear in the output panel and are also saved to the same directory as the script.

**From the command line:**

```bash
python test2.py
```

It'll ask you for the number of speakers, then run the full pipeline. Set the `INPUT_MP3` and `OUTPUT_DIR` environment variables to control which file it processes and where results go.

---

## Output files

After a successful run, you'll find these in the output directory:

| File | Contents |
|---|---|
| `meeting_minutes.txt` | The formatted minutes with all sections |
| `diarization_output.txt` | Raw speaker segments with timestamps |
| `asr_output.txt` | Transcribed text per segment with timestamps |
| `summarization.log` | Debug log from the summarization step |

---

## Notes

- The pipeline runs entirely on CPU. GPU is explicitly disabled in the code, so don't expect fast processing on long recordings — a one-hour meeting will take a while.
- Audio is normalised and filtered (low-pass at 3kHz, high-pass at 100Hz) before diarization. This helps with noisy recordings but may not suit all audio.
- The BART summarizer splits the transcript into ~900-token chunks, so very long meetings are handled incrementally.
- If the number of speakers you provide doesn't match reality, diarization accuracy will drop. It's used as a hard constraint (`min_speakers = max_speakers = num_speakers`).
