+++
date = '2026-06-10T14:00:00-04:00'
draft = false
title = 'Building Agentic Systems with Local Models'
tags = ['agents', 'local-LLM', 'AI', 'tool-use', 'open-source']
description = 'How to build autonomous AI agents that run entirely on your own hardware — tool calling, planning, and execution loops with local models.'
+++

## The Agent Shift

The LLM space pivoted hard toward agents over the past year. A model that can only chat is useful. A model that can call tools, browse the web, write files, and execute code is transformative. The challenge for the local AI community has been getting all of that capability without sending your orchestration to a cloud provider.

The good news: local agent systems are not only possible, they're increasingly competitive. A properly configured local agent running a Qwen 3.5 32B or Llama 4 8B can handle most of what people use cloud agents for — without the latency, cost, or privacy concerns.

## The Critical Ingredient: Tool Calling

The foundation of any agent is reliable tool calling. The model needs to output structured function calls that your runtime can parse and execute. Out of the box, most open-weight models support this through:

- **Built-in function calling** — models like Qwen 3.5 and Llama 4 are trained on tool-use formats and reliably output structured JSON
- **Custom prompt templates** — for models without native tool support, you can use grammars or constraint decoding (llama.cpp supports this with GBNF grammars)
- **JSON mode** — most local runtimes can force the model to output valid JSON, which you then parse for function names and arguments

I've tested tool-calling reliability across the major local models at 7–32B size. Qwen 3.5 14B is the most consistent for multi-step tool use, correctly formatting calls ~92% of the time in my testing. Llama 4 8B is close behind at ~88%. Both are good enough for production pipelines when combined with retry logic.

## Building a Simple Agent Loop

Here's the architecture I use for local agents — it works with any OpenAI-compatible runtime:

```
1. User sends a task
2. Agent formats the task with available tools as system prompt
3. Model generates a response (text or tool call)
4. If tool call: parse it, execute the function, return results to model
5. Model generates next action (tool call again, or final answer)
6. Repeat until done or max iterations reached
```

The loop is simple, but the key insight is that each iteration needs the full conversation history. Local models have context windows of 32K–128K tokens, which is plenty for most agent sessions. I rarely exceed 8K tokens even for complex multi-step tasks.

## Real Agent Examples

### Code Agent

I run a local agent that can edit files, run tests, and commit changes. It uses:
- Qwen 3.5 32B via Ollama (reasoning + tool calls)
- A set of tools: `read_file`, `write_file`, `patch`, `terminal`
- A safety guard: file writes are restricted to a project directory

This handles routine refactoring, bug fixing, and test writing without supervision. The latency is higher than a cloud agent (3–5 seconds per step vs 1–2), but I can run as many steps as I need without worrying about API costs.

### Research Agent

For research tasks that involve reading multiple documents and synthesising information:
- Llama 4 8B as the reasoning core (faster, good enough)
- Local RAG with ChromaDB for document retrieval
- A search tool that queries a local index of saved articles
- Output to a structured markdown report

Running entirely locally means I can throw 50 research papers at it and leave it running overnight — no API bill, no data leaving the house.

## Challenges and Workarounds

Local agent systems aren't without pain points:

| Challenge | Solution |
|-----------|----------|
| Token generation is slower | Use smaller models for simple tasks (4B–8B), only bring out 32B for complex reasoning |
| Context management | Implement sliding window summarisation for long-running agents; local context is a scarce resource |
| Reliability of multi-step plans | Add a verification step: the model self-critiques its output before executing destructive actions |
| Prompt sensitivity | Test your system prompt with 3–5 example tasks before going to production; small tweaks matter enormously |

## The Tool Ecosystem

A few projects making local agents practical right now:

- **LangChain / LangGraph** — works with any local model via OpenAI-compatible API; the agent framework is model-agnostic
- **CrewAI** — multi-agent orchestration that runs locally; great for role-based agent teams
- **Semantic Kernel** — Microsoft's agent framework, surprisingly good with local models via custom connectors
- **Built-from-scratch** — a simple agent loop in Python with `requests` against Ollama is ~100 lines and often works better than any framework

I've settled on a pattern of using simple custom agents for well-defined tasks and LangGraph for complex multi-agent workflows. The framework matters less than the model and prompt design.

## What's Coming

The next frontier is persistent memory for local agents — long-term storage that survives sessions and lets agents build cumulative knowledge. Projects like Mem0 and local implementations of MCP (Model Context Protocol) are making this practical. Within months, a locally-hosted agent will be able to maintain a weeks-long context, learn from its mistakes, and improve over time — all without phoning home.

Local agents aren't just a toy. They're a genuinely viable alternative to cloud-based agent platforms, with the added benefits of privacy, zero ongoing costs, and complete customisability. If you haven't tried building one yet, start with a simple tool-using loop and a 14B model. The capabilities will surprise you.
