# Setup Log

## 2026-04-05 — Initial Setup

### Environment
- Machine: Apple Silicon M4, 128 GB unified memory
- macOS: (fill in your version)
- Docker Desktop: (fill in version after install)
- VS Code: (fill in version)
- Continue extension: (fill in version)
- Ollama: (fill in version after install)

---

### Step 1 — Docker Desktop
- [ ] Installed Docker Desktop for Mac (Apple Silicon)
- [ ] Set conservative resource limits (RAM, CPU) in Docker Desktop preferences
- [ ] Verified: `docker --version`

---

### Step 2 — VS Code
- [ ] Installed VS Code
- [ ] Created dedicated profile: **AI Factory**
- [ ] Configured terminal, formatting, and extensions inside the profile

---

### Step 3 — Continue Extension
- [ ] Installed Continue from VS Code Marketplace
- [ ] Confirmed extension version
- [ ] Located or created `~/.continue/config.yaml` (or project-local config)
- [ ] Copied `ai-factory/continue/config.yaml` into active Continue config path

---

### Step 4 — Ollama (Native macOS)
- [ ] Downloaded Ollama from https://ollama.com
- [ ] Dragged app to /Applications
- [ ] Confirmed `ollama` is in PATH: `which ollama`
- [ ] Verified Ollama is running: `ollama list`

---

### Step 5 — Pull Gemma 4 Models
- [ ] Pulled main model: `ollama pull gemma4:31b`
- [ ] Pulled fast model: `ollama pull gemma4:e4b`
- [ ] Listed models: `ollama list`
- [ ] Noted approximate disk usage:

| Model | Size on disk |
|---|---|
| gemma4:31b | ~20 GB |
| gemma4:e4b | ~2.5 GB |

---

### Step 6 — Verify Ollama API
- [ ] Sent test request to `http://localhost:11434/api/chat`
- [ ] Received expected response: `Ollama and Gemma 4 are working.`
- [ ] Noted response latency (rough): ___

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "gemma4:31b",
  "messages": [
    {"role": "user", "content": "Reply with: Ollama and Gemma 4 are working."}
  ]
}'
```

---

### Step 7 — Configure Continue
- [ ] config.yaml applied with Gemma 4 31B and E4B
- [ ] Verified chat window appears in Continue sidebar
- [ ] Sent test message successfully to gemma4:31b via Continue
- [ ] Sent test message successfully to gemma4:e4b via Continue

---

### Step 8 — Open WebUI (Optional)
- [ ] Ran `docker compose up -d` from `ai-factory/docker/`
- [ ] Opened http://localhost:3000
- [ ] Confirmed Open WebUI connects to Ollama on host
- [ ] Tested a prompt via the web UI

---

## Notes and Observations

(Add anything unusual, performance notes, or configuration tweaks here.)

---

## Issues Encountered

(Track any problems and how you resolved them.)

| Date | Issue | Resolution |
|---|---|---|
| | | |
