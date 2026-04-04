# AI Factory

Local AI development environment for Apple Silicon M4 (128 GB unified memory).

**Stack:** VS Code + Continue + Ollama (native macOS) + Gemma 4 + Docker (side services only)

---

## Quick Start

### Prerequisites
- macOS (Apple Silicon)
- [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/)
- [VS Code](https://code.visualstudio.com)
- [Continue](https://marketplace.visualstudio.com/items?itemName=Continue.continue) (VS Code extension)
- [Ollama](https://ollama.com) (native macOS install — drag to Applications)

### 1. Pull the models
```bash
ollama pull gemma4:31b
ollama pull gemma4:e4b
ollama list
```

### 2. Verify Ollama
```bash
curl http://localhost:11434/api/chat -d '{
  "model": "gemma4:31b",
  "messages": [
    {"role": "user", "content": "Reply with: Ollama and Gemma 4 are working."}
  ]
}'
```

### 3. Apply Continue config
Copy `continue/config.yaml` to your active Continue config path, or point Continue to this file.

### 4. (Optional) Start Open WebUI
```bash
cd docker
docker compose up -d
# Open http://localhost:3000
```

---

## Project Layout

```
ai-factory/
├── continue/
│   ├── config.yaml          # Continue model config (Gemma 4 31B + E4B)
│   └── rules/
│       └── local-dev.md     # Coding rules loaded by Continue
├── docker/
│   └── docker-compose.yml   # Open WebUI (optional side service)
├── prompts/
│   ├── architecture.md      # System design prompt template
│   ├── refactor.md          # Safe refactoring prompt template
│   └── debug.md             # Systematic debugging prompt template
├── workspaces/
│   ├── app-one/             # First project workspace
│   └── app-two/             # Second project workspace
├── notes/
│   └── setup-log.md         # Step-by-step setup checklist and log
└── desktop-shell/
    └── README.md            # Future Tauri / Electron control plane (Phase 2+)
```

---

## Model Roles

| Model | Use for | Context |
|---|---|---|
| `gemma4:31b` | Deep reasoning, planning, code generation, code review | 8192 tokens |
| `gemma4:e4b` | Fast drafts, quick checks, short repetitive tasks | 4096 tokens |

---

## Key Principles

- **Ollama runs natively** on macOS. Do not run it inside Docker Desktop — GPU acceleration (Apple Metal) is not available there.
- **Docker is for helper services only** — Open WebUI, future vector stores, local APIs.
- **Start with short context** (4K–8K). Grow only when a workflow demands it.
- **Use the 31B model** for hard problems. Use E4B for fast iterations.
- **The future desktop app** (desktop-shell/) is a Phase 2+ control plane — not a VS Code replacement.

---

## Roadmap

| Phase | Goal |
|---|---|
| 1 | VS Code + Continue + native Ollama + Gemma 4 ✅ |
| 2 | Tauri/Electron shell for prompt libraries, model routing, run history |
| 3 | Containerized helper APIs, RAG services, team-facing local tools |

---

## References

- [Continue Install](https://docs.continue.dev/ide-extensions/install)
- [Continue + Ollama Guide](https://docs.continue.dev/guides/ollama-guide)
- [Continue config.yaml Reference](https://docs.continue.dev/reference)
- [Gemma 4 Model Card](https://ai.google.dev/gemma/docs/core/model_card_4)
- [Ollama Gemma 4 Library](https://ollama.com/library/gemma4)
- [Ollama macOS Install](https://docs.ollama.com/macos)
- [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/)
