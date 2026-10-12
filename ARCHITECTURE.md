# NIRMAN — System Architecture

## Overview

NIRMAN is built around a single, linear **voice pipeline** that connects four independent modules:

```
Microphone
    │
    ▼
┌─────────────────────┐
│   STT Engine        │  faster-whisper (local Whisper)
│   Whisper (local)   │  Converts spoken audio → text
└──────────┬──────────┘
           │  query text
           ▼
┌─────────────────────┐
│   RAG Pipeline      │  LangChain + ChromaDB
│   Retrieval         │  Embeds query → searches vector store
│   Augmented Gen.    │  Returns top-K relevant document chunks
└──────────┬──────────┘
           │  context + query
           ▼
┌─────────────────────┐
│   Local LLM         │  Ollama (Llama 3.2 / Mistral / Phi-3)
│   Ollama            │  Generates a grounded answer
└──────────┬──────────┘
           │  response text
           ▼
┌─────────────────────┐
│   TTS Engine        │  Piper TTS (offline, neural voices)
│   Piper TTS         │  Synthesises natural speech audio
└──────────┬──────────┘
           │
           ▼
        Speaker
```

---

## Module Breakdown

### 1. STT — Speech-to-Text (`src/stt/`)

| Component | Technology | Role |
|---|---|---|
| `audio_capture.py` | sounddevice + PyAudio | Records mic input, detects silence |
| `whisper_stt.py` | faster-whisper | Transcribes audio to text |

**Flow:**
1. Microphone stream opens via sounddevice.
2. Audio is captured in chunks; silence is detected by RMS energy threshold.
3. Once silence is detected, the buffer is flushed to faster-whisper.
4. Whisper returns a plain-text transcript.

**Model choices:**

| Model | Size | Speed | Accuracy |
|---|---|---|---|
| `tiny` | 75 MB | Fastest | Lower |
| `base` | 145 MB | Fast | Good (default) |
| `small` | 465 MB | Medium | Better |
| `medium` | 1.5 GB | Slow | High |
| `large` | 2.9 GB | Slowest | Best |

---

### 2. RAG Pipeline (`src/rag/`)

| Component | Technology | Role |
|---|---|---|
| `ingestion.py` | LangChain loaders | Loads PDF / DOCX / MD / TXT |
| `chunker.py` | RecursiveCharacterTextSplitter | Splits docs into 512-char chunks |
| `embedder.py` | nomic-embed-text (Ollama) | Converts chunks → dense vectors |
| `retriever.py` | ChromaDB | Stores vectors; retrieves top-K matches |
| `reranker.py` | Cross-encoder (optional) | Re-ranks retrieved chunks by relevance |

**Document Ingestion Flow:**
```
data/documents/
      │
      ▼  (PyMuPDF / Unstructured)
  Raw Text
      │
      ▼  (RecursiveCharacterTextSplitter, chunk=512, overlap=64)
  Chunks [ ][ ][ ][ ]
      │
      ▼  (nomic-embed-text via Ollama)
  Embeddings [0.12, -0.04, ...]
      │
      ▼
  ChromaDB  ── persisted to data/vectorstore/
```

**Query Flow:**
```
User query → embed query → cosine similarity search → top-5 chunks → build prompt
```

---

### 3. Local LLM (`src/llm/`)

| Component | Technology | Role |
|---|---|---|
| `ollama_client.py` | ollama Python SDK | Sends prompt to local Ollama server |
| `prompt_builder.py` | Python f-strings | Builds RAG-aware system prompts |

**Prompt Template:**
```
You are NIRMAN, a helpful AI assistant.
Answer based ONLY on the context below. If unsure, say so.

CONTEXT:
{retrieved_chunks}

HISTORY:
{conversation_history}

USER: {query}
NIRMAN:
```

**Supported models (via Ollama):**

| Model | VRAM | Best For |
|---|---|---|
| `phi3` | 4 GB | Fast responses, low hardware |
| `llama3.2` | 4 GB | Balanced (default) |
| `mistral` | 6 GB | Reasoning tasks |
| `llama3.1:70b` | 40 GB | Highest quality (GPU required) |

---

### 4. TTS — Text-to-Speech (`src/tts/`)

| Component | Technology | Role |
|---|---|---|
| `piper_tts.py` | Piper TTS | Synthesises speech from LLM response |
| `audio_player.py` | sounddevice + soundfile | Plays the generated WAV |

**Voice models (Piper):**

| Voice | Language | Quality |
|---|---|---|
| `en_US-lessac-medium` | English (US) | Medium (default) |
| `en_US-ryan-high` | English (US) | High quality |
| `en_GB-alan-medium` | English (UK) | Medium |
| `hi-IN-...` | Hindi | Available |

**Fallback chain:** Piper → pyttsx3 → print to console

---

## Data Flow Diagram

```
┌──────────┐     audio      ┌──────────────┐   text query   ┌────────────────┐
│   User   │──────────────▶│  Whisper STT │───────────────▶│  RAG Retriever │
│          │                └──────────────┘                 │  (ChromaDB)    │
│          │                                                  └───────┬────────┘
│          │                                                          │ top-K docs
│          │                                                          ▼
│          │                ┌──────────────┐  response text  ┌────────────────┐
│          │◀──────────────│  Piper TTS   │◀───────────────│  Ollama LLM    │
└──────────┘     speech     └──────────────┘                 └────────────────┘
```

---

## Component Interaction Map

```
nirman.py (CLI)
    │
    └── NirmanPipeline (src/core/pipeline.py)
            │
            ├── WhisperSTT      ──▶  captures + transcribes audio
            ├── RAGRetriever    ──▶  retrieves relevant chunks
            │       └── ChromaDB client
            ├── OllamaClient    ──▶  generates LLM response
            │       └── Ollama server (localhost:11434)
            ├── PiperTTS        ──▶  synthesises + plays speech
            └── ConversationSession  ──▶  maintains turn history
```

---

## Technology Choices — Rationale

| Decision | Choice | Why |
|---|---|---|
| LLM runtime | Ollama | Simple install, model management, REST API |
| STT | faster-whisper | 4x faster than original Whisper, same accuracy |
| TTS | Piper | Best quality/speed ratio for offline TTS |
| Vector DB | ChromaDB | Zero server setup, Python-native, persistent |
| Embeddings | nomic-embed-text | Best open-source embedding model, runs via Ollama |
| RAG framework | LangChain | Mature ecosystem, good ChromaDB integration |
| UI | Gradio | Zero-config browser UI ideal for AI demos |

---

## Hardware Requirements

| Config | RAM | GPU | Models |
|---|---|---|---|
| Minimal | 8 GB | None | phi3 + Whisper tiny |
| Recommended | 16 GB | None | llama3.2 + Whisper base |
| GPU (fast) | 16 GB | 8 GB VRAM | mistral + Whisper small |
| Power | 32 GB | 24 GB VRAM | llama3.1:70b + Whisper large |

---

## Security & Privacy

- **All computation is local** — no data sent to any server
- **No telemetry** — nothing phoned home
- **Documents stay on disk** — ChromaDB persists to `data/vectorstore/`
- **Credentials** — only `config.yaml`, no API keys needed
