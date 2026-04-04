# code-with-pq — Local AI Factory

> **Local-first AI coding environment** — VS Code + Continue + native Ollama + Gemma 4 + Docker side services.  
> Built for Apple Silicon M-series Macs. No cloud dependency. No vendor lock-in.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [System Requirements](#system-requirements)
   - [RAM Requirements](#ram-requirements)
   - [Disk Space Requirements](#disk-space-requirements)
   - [macOS Requirements](#macos-requirements)
3. [Stack at a Glance](#stack-at-a-glance)
4. [Model Size Reference](#model-size-reference)
5. [Setup Steps](#setup-steps)
   - [Step 1 — Install Docker Desktop](#step-1--install-docker-desktop)
   - [Step 2 — Install VS Code](#step-2--install-vs-code)
   - [Step 3 — Install Continue](#step-3--install-continue)
   - [Step 4 — Install Ollama natively on macOS](#step-4--install-ollama-natively-on-macos)
   - [Step 5 — Pull Gemma 4 models](#step-5--pull-gemma-4-models)
   - [Step 6 — Verify Ollama](#step-6--verify-ollama)
   - [Step 7 — Configure Continue](#step-7--configure-continue)
   - [Step 8 — Start Docker side services](#step-8--start-docker-side-services)
6. [RAM Tier Guide](#ram-tier-guide)
7. [Model Selection by Mac Memory](#model-selection-by-mac-memory)
8. [Project Structure](#project-structure)
9. [Performance Tuning](#performance-tuning)
10. [Troubleshooting](#troubleshooting)
11. [References](#references)

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    macOS Host                           │
│                                                         │
│  ┌──────────────┐    ┌──────────────────────────────┐  │
│  │   VS Code    │◄──►│  Continue (AI Extension)     │  │
│  │  (Editor)    │    │  Chat / Plan / Agent modes   │  │
│  └──────────────┘    └──────────────┬───────────────┘  │
│                                     │ localhost:11434   │
│                              ┌──────▼──────────┐        │
│                              │  Ollama (native) │        │
│                              │  Metal GPU accel │        │
│                              └──────┬───────────┘        │
│                                     │                    │
│                    ┌────────────────▼─────────────────┐ │
│                    │  Gemma 4 Models                  │ │
│                    │  gemma4:e4b  (9.6 GB) — fast     │ │
│                    │  gemma4:31b  (20 GB)  — deep     │ │
│                    └──────────────────────────────────┘ │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Docker Desktop (side services only)            │   │
│  │  Open WebUI · Postgres · Qdrant vector store    │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

**Key principle:** Ollama runs **natively** on the macOS host so it can use **Metal GPU acceleration**. Docker is the support layer — not the inference engine.

---

## System Requirements

### ⚠️ RAM Requirements

This is the single most important constraint. Model inference is almost entirely memory-bound.

| Your RAM | Minimum Usable Model | Recommended Model | Notes |
|---|---|---|---|
| **8 GB** | Not recommended | — | Cannot run a meaningful local AI stack |
| **16 GB** | `gemma4:e2b` (7.2 GB) | `gemma4:e2b` | Keep Docker closed; short context only |
| **24 GB** | `gemma4:e4b` (9.6 GB) | `gemma4:e4b` | Good travel/compact desktop tier |
| **32 GB** | `gemma4:e4b` | `gemma4:e4b` + `gemma4:26b` | Best value tier — productive local setup |
| **64 GB** | `gemma4:26b` (18 GB) | `gemma4:26b` + `gemma4:31b` | Serious workstation tier |
| **128 GB** (target) | `gemma4:31b` (20 GB) | `gemma4:31b` + `gemma4:26b` | Full workstation — runs everything easily |

> **This repo is tuned for Apple Silicon M4 with 128 GB unified memory.** At 128 GB, memory is never the bottleneck — context length, latency, and workflow design are what you optimise instead.

**DDR generation also matters:**

| Generation | Same RAM | Experience |
|---|---|---|
| DDR3 | 32 GB | Older platform CPU/bus limits throughput even if model fits |
| DDR4 | 32 GB | Materially better; paired with newer CPU + NVMe |
| Unified (Apple silicon) | 32 GB+ | Best: coherent memory, Metal acceleration, no VRAM separation |

---

### Disk Space Requirements

| Item | Approximate Size |
|---|---|
| `gemma4:e2b` model blob | ~7.2 GB |
| `gemma4:e4b` model blob | ~9.6 GB |
| `gemma4:26b` model blob | ~18 GB |
| `gemma4:31b` model blob | ~20 GB |
| Ollama app + CLI | ~600 MB |
| VS Code + Continue | ~500 MB |
| Docker Desktop | ~1.5 GB app + images |
| Open WebUI Docker image | ~1–2 GB |
| Postgres + Qdrant images | ~1 GB combined |
| **Working headroom** | **≥ 20 GB free** |

> **Recommended minimum free disk: 80–100 GB** on a fast internal SSD.  
> If you pull all four Gemma 4 sizes, plan for at least **60 GB for models alone**.  
> Ollama stores models at `~/.ollama` by default — point it to a larger drive before pulling if needed.

---

### macOS Requirements

| Requirement | Value |
|---|---|
| macOS version | Current + previous two major releases (Docker Desktop requirement) |
| Architecture | Apple Silicon (M1/M2/M3/M4) — native Metal path |
| Rosetta 2 | Recommended for Docker Desktop (`softwareupdate --install-rosetta`) |
| Internet | Only needed during initial downloads; offline after that |

---

## Stack at a Glance

| Role | Tool | Why |
|---|---|---|
| Editor | **VS Code** | Full IDE — tabs, terminal, Git, debugger, extensions |
| AI extension | **Continue** | Chat, Plan, Agent modes directly in the editor |
| Model runtime | **Ollama** (native macOS) | Apple Metal GPU acceleration; `localhost:11434` |
| Primary model | **`gemma4:31b`** | 256K context; deep reasoning and code generation |
| Fast model | **`gemma4:e4b`** | Quick drafts, triage, fast iteration loops |
| Side services | **Docker Desktop** | Databases, vector stores, optional web UI |

---

## Model Size Reference

| Model | Blob size | Context window | Best use |
|---|---|---|---|
| `gemma4:e2b` | ~7.2 GB | 256K | Fallback / travel / low-RAM machines |
| `gemma4:e4b` | ~9.6 GB | 256K | Default all-round local coding assistant |
| `gemma4:26b` | ~18 GB | 256K | Deeper reasoning on stronger machines |
| `gemma4:31b` | ~20 GB | 256K | Heaviest local option; use for hard tasks |

> Model file size ≠ total working memory. Runtime overhead and context cache add on top. A 20 GB model may consume 24–28 GB at runtime depending on context length in use.

---

## Setup Steps

### Step 1 — Install Docker Desktop

1. Download **Docker Desktop for Mac (Apple silicon)** from [docker.com](https://docs.docker.com/desktop/setup/install/mac-install/).
2. Install Rosetta 2 first if prompted (or run it manually):
   ```bash
   softwareupdate --install-rosetta
   ```
3. Open Docker Desktop and complete onboarding.
4. **Set conservative memory limits** in Docker Desktop → Settings → Resources. Start with 8 GB max so containers do not compete with Ollama inference.
5. Keep Docker Desktop **closed when not actively using containers** on lower-memory Macs.

> Do **not** run Ollama inside Docker on macOS — it loses Apple Metal GPU acceleration. Docker is for side services only.

---

### Step 2 — Install VS Code

1. Download from [code.visualstudio.com](https://code.visualstudio.com/).
2. Create a **dedicated VS Code profile** for AI factory work:
   - Open Command Palette → `Profiles: Create Profile…`
   - Name it `AI Factory`
   - This isolates your AI extensions, settings, and model config from other projects.

---

### Step 3 — Install Continue

1. Open VS Code → Extensions (`⇧⌘X`).
2. Search for **Continue** and install it.
3. Continue uses a local `config.yaml` for all configuration — no cloud sign-in required.
4. The config lives at `ai-factory/continue/config.yaml` in this repo (see [Step 7](#step-7--configure-continue)).

---

### Step 4 — Install Ollama natively on macOS

1. Download the **Ollama macOS app** from [ollama.com](https://ollama.com/).
2. Drag `Ollama.app` into your Applications folder.
3. Launch the app — it installs the CLI and starts the background service.
4. Verify the CLI is available:
   ```bash
   ollama --version
   ollama list
   ```
5. Ollama serves the API at `http://localhost:11434` once running.

> If you want to move model storage from the default `~/.ollama`, configure a custom path **before** pulling large models.

---

### Step 5 — Pull Gemma 4 models

Pull the models you need based on your RAM tier (see [RAM Requirements](#-ram-requirements)):

```bash
# Fast helper model — recommended for everyone (9.6 GB)
ollama pull gemma4:e4b

# Primary deep reasoning model — recommended for 32 GB+ RAM (20 GB)
ollama pull gemma4:31b

# Optional: smaller fallback (7.2 GB)
ollama pull gemma4:e2b

# Optional: workstation step-up model (18 GB)
ollama pull gemma4:26b

# Verify pulls
ollama list
```

**On a 128 GB machine, pulling all four is fine.** On 32 GB, start with just `e4b` + `26b`.

---

### Step 6 — Verify Ollama

Test that Ollama is responding before you touch Continue:

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "gemma4:e4b",
  "messages": [
    {"role": "user", "content": "Reply with: Ollama and Gemma 4 are working."}
  ]
}'
```

You should see a streamed JSON response. If you get a connection error, relaunch the Ollama app.

---

### Step 7 — Configure Continue

The config is already in this repo at `ai-factory/continue/config.yaml`. To activate it:

1. Open Continue's settings in VS Code.
2. Point Continue at your local config file path, or symlink `~/.continue/config.yaml` to this repo's config:
   ```bash
   mkdir -p ~/.continue
   ln -sf "$(pwd)/ai-factory/continue/config.yaml" ~/.continue/config.yaml
   ```
3. Reload the VS Code window (`⇧⌘P` → `Developer: Reload Window`).

The config sets up:

| Model name | Ollama model | Role | Context length |
|---|---|---|---|
| Gemma 4 31B | `gemma4:31b` | chat, edit, apply | 8192 |
| Gemma 4 E4B Fast | `gemma4:e4b` | chat | 4096 |

> Start with **4K–8K context** for stability. Expand only when a specific workflow needs longer context — even though Gemma 4 supports 256K, large contexts slow down responses significantly.

**Continue modes:**
- **Chat** — explanation, design discussion, Q&A
- **Plan** — safe inspection mode before making edits
- **Agent** — model can propose file edits and run terminal commands (with your approval)

---

### Step 8 — Start Docker side services

```bash
cd ai-factory/docker
docker compose up -d
docker ps
```

This starts **Open WebUI** at [http://localhost:3000](http://localhost:3000) — a second interface for quick prompt experiments and demos, connected to your local Ollama instance.

Open WebUI is optional. VS Code + Continue is the primary workflow surface.

---

## RAM Tier Guide

### DDR4 / Apple Silicon

| RAM | Practical fit | VS Code + Continue | Docker sidecars | Verdict |
|---|---|---|---|---|
| 8 GB | E2B only and cramped | Basic chat only | No | Bare minimum |
| 16 GB | E2B good, E4B borderline | Realistic starter if disciplined | Light only | Starter tier |
| 32 GB | E4B sweet spot, 26B limited | Productive for most workflows | Yes | **Best value tier** |
| 64 GB | E4B excellent, 26B solid, 31B reasonable | Strong workstation feel | Yes | Serious local tier |
| 128 GB | Anything in the family | Full workflow, long contexts | Yes, freely | **Top tier** |

### DDR3 (older machines)

| RAM | Practical fit | Notes |
|---|---|---|
| 8 GB | Avoid | Not enough headroom for AI stack + editor + containers |
| 16 GB | E2B only | Short context, no Docker, patience required |
| 32 GB | E2B good, E4B possible | Workable but older platform limits throughput |
| 64 GB | E4B good, 26B experimental | Capacity helps, but old-platform latency is still a factor |

---

## Model Selection by Mac Memory

| Mac memory tier | Best starting model | Second model to add |
|---|---|---|
| 16 GB | E2B or E4B | None — start lean |
| 24 GB | E4B | 26B only if you accept slower runs |
| 32 GB | E4B | 26B |
| 64 GB+ | 26B | 31B |
| 128 GB (this machine) | 31B | E4B (as fast helper) |

---

## Project Structure

```
.
├── .gitignore                    # Hides .docx files and system artifacts
├── README.md                     # This file
├── ai-factory/
│   ├── continue/
│   │   ├── config.yaml           # Continue AI extension configuration
│   │   └── rules/
│   │       └── local-dev.md      # Local dev rules for the AI agent
│   ├── docker/
│   │   └── docker-compose.yml    # Open WebUI + optional Postgres + Qdrant
│   ├── prompts/
│   │   ├── architecture.md       # Architecture prompt templates
│   │   ├── refactor.md           # Refactor prompt templates
│   │   └── debug.md              # Debug prompt templates
│   ├── workspaces/
│   │   ├── app-one/
│   │   └── app-two/
│   ├── notes/
│   │   └── setup-log.md          # Personal setup notes
│   └── desktop-shell/            # Future Tauri/Electron control plane
```

---

## Performance Tuning

- **Start with short context.** Use 4K–8K for daily work. Grow only when a task actually needs it.
- **Two-model pattern.** Use `e4b` for fast drafts and triage. Use `31b` for planning, architecture, and hard code tasks.
- **Docker memory limits.** Cap Docker at 8 GB in settings so it does not compete with Ollama.
- **Close unused containers.** On lower-memory Macs, stop Docker when not using side services.
- **Preload the model.** Send a warm-up request to Ollama before a long work session:
  ```bash
  curl http://localhost:11434/api/chat -d '{"model":"gemma4:31b","messages":[{"role":"user","content":"ping"}]}'
  ```
- **Memory errors in Continue?** Lower `contextLength` in `config.yaml` first — that is almost always the fix.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `ollama: command not found` | Relaunch `Ollama.app` and allow CLI link creation |
| Model calls are very slow | Switch to `e4b` first; lower `contextLength` |
| Docker Desktop eating RAM | Stop unused containers; close Docker when idle |
| Continue changes not visible | Reload VS Code window: `⇧⌘P` → `Developer: Reload Window` |
| Open WebUI can't reach Ollama | Verify Ollama is running: `curl http://localhost:11434` |
| Context window errors | Reduce `contextLength` in `config.yaml`; restart Continue |

---

## References

| Resource | URL |
|---|---|
| Continue — Install | https://docs.continue.dev/ide-extensions/install |
| Continue — Ollama guide | https://docs.continue.dev/guides/ollama-guide |
| Continue — config.yaml reference | https://docs.continue.dev/reference |
| Ollama — macOS install | https://docs.ollama.com/macos |
| Ollama — Gemma 4 library | https://ollama.com/library/gemma4 |
| Ollama — API introduction | https://docs.ollama.com/api/introduction |
| Ollama — GPU / hardware support | https://docs.ollama.com/gpu |
| Google — Gemma 4 model card | https://ai.google.dev/gemma/docs/core/model_card_4 |
| Google — Gemma thinking mode | https://ai.google.dev/gemma/docs/capabilities/thinking |
| Google — Run Gemma with Ollama | https://ai.google.dev/gemma/docs/integrations/ollama |
| Docker Desktop for Mac | https://docs.docker.com/desktop/setup/install/mac-install/ |
| Docker — host.docker.internal | https://docs.docker.com/desktop/features/networking/networking-how-tos/ |

---

Source basis verified April 2026. Built for Apple Silicon M4 · 128 GB unified memory.
