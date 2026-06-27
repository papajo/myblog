+++
date = '2026-06-24T16:30:00-04:00'
draft = false
title = 'Practical Tips for Setting Up Your Own AI Stack'
tags = ['beginner', 'local-LLM', 'setup', 'open-source', 'hardware']
description = 'A no-nonsense guide to getting your local AI stack running — hardware picks, software choices, and configuration that actually works.'
+++

## You Already Have What You Need

The most common misconception about running AI locally is that you need expensive hardware. You probably don't. The vast majority of useful local AI work can be done on hardware you already own — you just need to know which models to run and how to configure things.

This guide skips the theory and gets straight to what works, based on setting up stacks on half a dozen machines with different specs.

## Step 1: Pick Your Runtime

You need one piece of software that downloads models, manages them, and serves them. Here are your options by experience level:

| Runtime | Best For | Why |
|---------|----------|-----|
| **Ollama** | Everyone starting out | Two commands to a running model. Apple Silicon, NVIDIA, AMD, CPU — all supported. OpenAI-compatible API included. |
| **LM Studio** | GUI lovers | Chat interface, model browser, config sliders. No terminal required. |
| **llama.cpp** | Advanced users | Full control over every parameter. GGUF quantisations. Best performance. |
| **LocalAI** | Docker-centric setups | Container-native, supports more than just text (whisper, stable diffusion). |

**Recommendation:** Start with Ollama. It handles the 90% case and you can always swap runtimes later.

## Step 2: Choose Your First Model

This is where most people overthink it. Pick one model and get it running. The specific model matters far less than just starting.

- **Mac with 8GB RAM** → `ollama run qwen3.5:4b` (fast, multilingual, capable)
- **Mac with 16GB RAM** → `ollama run llama3.2` or `ollama run qwen3.5:7b`
- **Mac with 24–36GB RAM** → `ollama run qwen3.5:14b` (the sweet spot)
- **Windows/Linux with NVIDIA GPU** → same models, but expect 2–3x speed on GPU
- **Any machine, minimal RAM** → `ollama run phi3:3.8b` or `ollama run gemma2:2b`

**Don't** start with a 70B model. You'll be frustrated by the speed, tempted to buy hardware you don't need, and likely give up. A 7B–14B model on modern hardware gives you 95% of the utility for 10% of the resource cost.

## Step 3: Get a Chat Interface

Raw terminal chat gets old fast. Add a front-end:

```bash
docker run -d -p 3000:8080 \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  ghcr.io/open-webui/open-webui:main
```

Open WebUI gives you a ChatGPT-like experience with your local models — conversations, file uploads, RAG, and multi-user support. It's the single best companion to Ollama.

Alternatively, if you use VS Code, install **Continue.dev**. It turns your editor into a local AI coding assistant that uses whatever model you have running in Ollama. This alone justifies setting up the stack.

## Step 4: Configure for Performance

A few tweaks that make a real difference:

**Quantisation matters more than model size.** A Q4_K_M quantisation of a 14B model runs on 8GB RAM and is nearly indistinguishable from the full-precision version. A Q2 quant of a 70B model will feel dumb compared to a Q8 quant of a 7B model. Rule of thumb: prefer smaller models at higher precision.

**Context length is negotiable.** The default 2048-token context in many runtimes is too small. Set `num_ctx` to 8192 or 16384 — the quality improvement is noticeable. The memory cost is roughly 0.5–1 GB per 8K of context, so keep an eye on it.

**Batch inference for throughput.** If you're processing many prompts (batch summarisation, bulk classification), increase `num_batch` to 512. This uses more VRAM but can double your throughput.

**Keep the model warm.** Loading a model takes 1–3 seconds. If you're doing interactive work, set `keep_alive` to a longer duration in Ollama so it stays loaded in memory between requests.

## Step 5: Set Up a Basic RAG Pipeline

Retrieval-Augmented Generation is the single most impactful upgrade to a local AI stack. It lets your model answer questions about your own documents.

Minimal setup:

```bash
# Install ChromaDB
pip install chromadb

# Pull an embedding model
ollama pull nomic-embed-text

# In Python:
import chromadb
client = chromadb.Client()
collection = client.create_collection("my-docs")
collection.add(
    documents=["Your text here"],
    ids=["doc1"]
)
# Query:
results = collection.query(query_texts=["What is this about?"], n_results=3)
```

Pass the retrieved context to your model alongside the question, and suddenly your local LLM knows about your specific documents, codebase, or knowledge base.

## Step 6: Automate Daily Use

The stack is most valuable when it's integrated into your routine. Here's what I do:

- **Always-on service:** Ollama runs as a login item on my Mac. It's always available, using negligible resources when idle.
- **Keyboard shortcut:** I have a Raycast script that sends selected text to the local model and replaces it with the response — for instant translation, summarisation, or rewrite.
- **Daily agent:** A cron job runs a local agent every morning to process my overnight notes and prepare a daily briefing. All local, all automated.

## The One Thing Not to Skip

Whatever you do, set up **aliases**. Even experienced users forget the exact commands:

```bash
alias ai='ollama run qwen3.5:14b'
alias ai-fast='ollama run qwen3.5:7b'
alias ai-code='ollama run llama3.2'
```

The less friction between you and the model, the more you'll use it. And using it regularly is the only way to figure out what actually works for your workflow.

## Final Advice

The perfect setup doesn't exist. Start simple — Ollama + Open WebUI + one model — and iterate from there. Run it for a week. Notice what frustrates you. Fix that one thing. Repeat. Within a month you'll have a stack that's genuinely useful and entirely yours.

The barrier to entry has never been lower. Your hardware is ready. Your tools are free. Go set it up.
