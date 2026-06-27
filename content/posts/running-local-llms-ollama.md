+++
date = '2026-04-15T10:00:00-04:00'
draft = false
title = 'Running Local LLMs with Ollama and Custom Toolchains'
tags = ['local-LLM', 'Ollama', 'open-source', 'tooling']
description = 'A practical walkthrough of getting started with Ollama, pulling models, and integrating local LLMs into your daily workflow with custom toolchains.'
+++

## Why Ollama?

If you've been curious about running large language models on your own machine but bounced off the complexity of llama.cpp build flags, Python environment hell, or CUDA configuration, Ollama is the answer. It wraps all of that into a clean command-line interface that just works — on macOS, Linux, and Windows.

```bash
ollama pull llama3.2
ollama run llama3.2
```

That's it. You're now running a reasonably capable 3B parameter model on your laptop. No Docker, no Python, no GPU driver struggle. It's the `brew install` moment for local AI.

## What's Under the Hood

Ollama is essentially a well-engineered wrapper around llama.cpp with a model registry, prompt template management, and an OpenAI-compatible API server baked in. When you `ollama pull`, it downloads a layered model bundle — weights, template, configuration — and quantises them for efficient CPU and GPU inference.

The models are stored in `~/.ollama/models/blobs/` and you can inspect, modify, or build your own from a Modelfile. A Modelfile is like a Dockerfile for LLMs:

```dockerfile
FROM llama3.2
PARAMETER temperature 0.7
PARAMETER top_p 0.9
SYSTEM "You are a helpful coding assistant."
```

Running `ollama create my-coder -f ./Modelfile` builds a custom model ready to use.

## Toolchains Beyond Chat

The real power of Ollama isn't the chat interface — it's the API. Every running model exposes an endpoint at `http://localhost:11434/v1/` that's compatible with the OpenAI SDK. This means any tool that speaks OpenAI's API can talk to your local model with a one-line config change.

I use this with:
- **Continue.dev** — VS Code and JetBrains plugin for AI-assisted coding. Point it at Ollama and you get inline completions and chat without sending code to a third party.
- **Aider** — terminal-based AI pair programming. With `--model ollama/qwen3.5`, it runs entirely locally.
- **Open WebUI** — a self-hosted ChatGPT-like interface that connects to Ollama and supports multi-user, RAG, and tool use.
- **Hermes (this agent)** — my own assistant platform has Ollama as a provider. Switching between cloud and local is one command: `/model ollama/qwen3.5:7b`.

## Custom Toolchains: Connecting the Pieces

A custom toolchain means wiring Ollama into your existing ecosystem. Here's a simple pipeline I run for local document analysis:

1. A script watches a directory for new PDFs
2. On arrival, it extracts text with `pdftotext` and chunks it
3. Each chunk is embedded using `ollama pull nomic-embed-text` and stored in ChromaDB
4. A query script retrieves relevant chunks and sends them to `ollama run llama3.2` with a prompt instructing summarisation

All of this runs on a $0/hour laptop, no API keys, no data leaving the machine.

## Hardware Reality Check

You don't need a datacenter. Here's what works today:

| Hardware | Model Size | Tokens/sec |
|----------|-----------|------------|
| M1 Mac, 8GB | 3B–7B (Q4) | 15–25 |
| M2 Mac, 16GB | 7B–14B (Q4) | 20–35 |
| RTX 3090, 24GB | 13B–33B (Q4) | 40–60 |
| M3 Max, 64GB | 70B (Q4) | 8–12 |

The M-series Macs punch way above their weight thanks to unified memory. A 7B quantised model uses about 4–6 GB of RAM and runs at a comfortable reading speed on any Apple Silicon machine.

## Getting Started Today

If you're on macOS: `brew install ollama` and you're minutes away from your first local model. Linux is just a `curl` script. Once it's running, explore the model library at [ollama.com/library](https://ollama.com/library) — there are hundreds of models from tiny 1B code models up to 70B reasoning beasts.

The barrier to local AI has never been lower. The hardware you already own is probably enough. Go pull a model and see what's possible.
