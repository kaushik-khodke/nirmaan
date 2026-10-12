<div align="center">

# 🧠 NIRMAN — निर्माण

**N**eural **I**ntelligent **R**easoning **M**achine with **A**ugmented k**N**owledge

*A fully offline, privacy-first AI voice assistant — Local LLM + RAG + STT + TTS*

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)
![Ollama](https://img.shields.io/badge/LLM-Ollama-FF6B35?style=flat-square)
![Whisper](https://img.shields.io/badge/STT-Whisper-00B2FF?style=flat-square)
![ChromaDB](https://img.shields.io/badge/VectorDB-ChromaDB-7C3AED?style=flat-square)
![Piper](https://img.shields.io/badge/TTS-Piper-06B6D4?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)

> **NIRMAN** (निर्माण) means *"Creation"* in Sanskrit — an AI that creates answers from your own knowledge.

</div>

---

## What is NIRMAN?

NIRMAN is a **fully local, privacy-first AI assistant** that runs 100% on your machine.
You speak to it → it searches your documents → it answers in your voice → no cloud, no API keys, no data leaving your machine.

### Core Pipeline

```
🎤 Voice  →  Whisper STT  →  ChromaDB RAG  →  Ollama LLM  →  Piper TTS  →  🔊 Speech
```

---

## Features

| Feature | Description |
|---|---|
| 🔒 **100% Offline** | No internet required after setup |
| 🎤 **Voice Input** | Whisper transcribes speech accurately, locally |
| 📚 **Document Q&A** | Index PDFs, DOCX, Markdown, TXT files |
| 🧠 **RAG Pipeline** | Answers grounded in *your* documents |
| 🔊 **Voice Output** | Natural TTS responses via Piper |
| 💬 **Chat Mode** | Text interface also available |
| 📎 **Source Citations** | Every answer links back to the source document |
| 🔄 **Hot Reload** | Add new documents without restarting |

---

## Tech Stack

| Layer | Tool | Notes |
|---|---|---|
| **Local LLM** | [Ollama](https://ollama.ai) | Runs Llama 3.2, Mistral, Phi-3 locally |
| **STT** | [Whisper](https://github.com/openai/whisper) (faster-whisper) | Best offline speech recognition |
| **TTS** | [Piper TTS](https://github.com/rhasspy/piper) | Fast, natural, fully offline |
| **Embeddings** | nomic-embed-text (via Ollama) | Local embedding model |
| **Vector DB** | [ChromaDB](https://www.trychroma.com/) | Persistent local vector store |
| **RAG Framework** | [LangChain](https://langchain.com/) | Orchestrates the RAG pipeline |
| **Document Parsing** | PyMuPDF + Unstructured | PDF, DOCX, HTML, MD support |
| **Web UI** | [Gradio](https://gradio.app/) | Clean, local browser interface |

---

## Quick Start

### 1. Prerequisites

- Python 3.11+
- [Ollama](https://ollama.ai/download) installed and running
- 8 GB RAM minimum (16 GB recommended for large models)
- A microphone (for voice mode)

### 2. Install

```bash
git clone https://github.com/yourname/nirman.git
cd nirman
pip install -r requirements.txt
```

### 3. Pull a local model

```bash
ollama pull llama3.2          # ~2 GB, fast (recommended)
ollama pull nomic-embed-text  # embedding model (required for RAG)
```

### 4. Ingest your documents

```bash
# Drop PDFs / TXTs / MDs into data/documents/
python nirman.py ingest
```

### 5. Run

```bash
python nirman.py voice   # 🎤 Voice mode
python nirman.py chat    # 💬 Text mode
python nirman.py ui      # 🌐 Web UI (Gradio)
```

---

## Project Structure

```
nirman/
├── src/
│   ├── core/           # Pipeline orchestration & config
│   ├── stt/            # Whisper speech-to-text
│   ├── tts/            # Piper text-to-speech
│   ├── llm/            # Ollama LLM client
│   └── rag/            # ChromaDB RAG pipeline
├── data/
│   ├── documents/      # 📂 Drop your documents here
│   └── vectorstore/    # ChromaDB persisted store
├── models/             # Piper voice models
├── README.md
├── ARCHITECTURE.md     # Detailed system architecture
├── requirements.txt
└── config.yaml         # All settings in one place
```

---

## Configuration

All settings live in `config.yaml`:

```yaml
llm:
  model: "llama3.2"        # any Ollama model
stt:
  model: "base"            # tiny | base | small | medium | large
tts:
  voice: "en_US-lessac-medium"
rag:
  top_k: 5                 # chunks retrieved per query
  chunk_size: 512
```

---

## Roadmap

- [x] Core voice pipeline (STT → RAG → LLM → TTS)
- [ ] Web UI (Gradio)
- [ ] Multi-language support (Hindi, French, etc.)
- [ ] GPU acceleration (CUDA)
- [ ] Conversation memory persistence
- [ ] REST API mode
- [ ] Docker container

---

## License

MIT — free to use, modify, and distribute.

---

<div align="center"><i>Built with ❤️ — Fully local, fully private, fully yours.</i></div>
