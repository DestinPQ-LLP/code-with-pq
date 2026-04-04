Local AI Factory Setup Plan
Recommended stack for Apple Silicon M4 (128 GB unified memory): VS Code + Continue + Ollama + Gemma 4 + Docker-assisted local services
Prepared for local-first coding, reasoning, and future desktop-app orchestration.
Bottom line: on macOS, the best architecture is native Ollama for model inference + Docker Desktop for optional side services. Running Ollama itself in Docker Desktop on a Mac is possible, but it will not use Apple GPU acceleration, so it is not the recommended daily setup.

Machine profile assumed: Apple Silicon M4 laptop with 128 GB unified memory. This hardware is strong enough to run Gemma 4 workstation-class local models and still leave room for VS Code, terminals, browsers, containers, and project workspaces.
1. Executive Recommendation
Use VS Code as the editor, Continue as the AI extension layer, Ollama as the local model runtime, Gemma 4 as the primary reasoning model, and Docker Desktop only for supporting services and future packaging. This gets you productive quickly without wasting months rebuilding an IDE shell before your AI workflows are useful.
• Primary editor: VS Code
• Primary AI extension: Continue
• Primary local runtime: Ollama running natively on macOS
• Primary model: gemma4:31b for deep reasoning, planning, and code generation
• Fast helper model: gemma4:e4b for lighter prompts and quick checks
• Docker role on Mac: helper services, optional web UI, databases, future local APIs, packaging—not primary Gemma 4 inference
• Future desktop app path: build a thin Tauri or Electron control plane around this stack after the workflows are proven
2. Architecture Decision Matrix
Option	What it means	On your Mac	Verdict
A. Recommended	Native Ollama + Gemma 4; VS Code + Continue; Docker only for side services	Fastest and most sensible	Use this
B. Full Docker	Run Ollama itself inside Docker Desktop on macOS	Works, but no Apple GPU acceleration	Only for testing
C. Build your own IDE first	Create a full VS Code-like desktop app before workflows are stable	Slowest path to value	Do later

Why this recommendation matters: Ollama explicitly states that GPU acceleration in Docker is available on Linux and Windows (with WSL2), but not on Docker Desktop for macOS because GPU passthrough and emulation are not available there. Ollama also documents that Apple devices use Metal for GPU acceleration when Ollama runs directly on the host.
3. Minimum Requirements and Practical Targets
Layer	Minimum	Recommendation for your machine
macOS / Docker Desktop	Supported macOS release; Docker Desktop needs at least 4 GB RAM	You exceed this easily; keep Docker resources controlled
Disk space	Enough for models and images	Reserve 80–150 GB practical working room
Editor	VS Code	Use a dedicated profile for AI work
AI extension	Continue	Install from Marketplace; use local YAML config
Model runtime	Ollama	Run natively on macOS for Metal acceleration
Primary model	One local model	gemma4:31b as the main reasoning model
Fast model	Optional	gemma4:e4b for quicker, cheaper-feeling local responses

Model sizing note: Ollama’s Gemma 4 library page lists gemma4:26b at roughly 18 GB and gemma4:31b at roughly 20 GB, both with a 256K context window. On your 128 GB machine, the practical bottleneck is not whether the model fits, but how much total memory and context length you want to reserve for the rest of your workflow.
4. Architecture Overview
1. User / IDE Layer — VS Code workspace, terminal, Git, debugger, source control, tasks
2. AI Extension Layer — Continue handles chat, edit/apply flows, agent behavior, local config, and optional MCP tools
3. Local Runtime Layer — Ollama runs on the Mac host and serves models over http://localhost:11434
4. Model Layer — gemma4:31b for deep reasoning + optional gemma4:e4b for fast tasks
5. Container Layer — Docker Desktop runs optional side services such as a web UI, future local APIs, vector stores, and later packaging

Interpretation: Continue is the workflow brain inside the editor, but Ollama is the actual inference engine. Docker is not the AI engine on your Mac; Docker is the surrounding service layer.
5. Step-by-Step Setup
Step 1 — Install Docker Desktop
Install Docker Desktop for Mac (Apple silicon). Docker documents that Docker Desktop for Mac supports the current and previous two major macOS releases, and that the minimum RAM requirement is 4 GB. On your machine, set conservative Docker resource limits at first so containers do not steal memory from local model inference.
• Use Docker Desktop mainly for helper services.
• Do not make Docker the default place where Gemma 4 runs on macOS.
• Keep Docker’s disk image on a drive with plenty of free space.
Step 2 — Install VS Code
Install VS Code normally. Create a dedicated profile for your AI factory work so your model, extension, terminal, formatting, and workspace settings stay isolated from other development profiles.
Step 3 — Install Continue
Install Continue from the VS Code Marketplace. Continue’s current configuration model is YAML-based. Its config.yaml supports models, rules, prompts, docs, and mcpServers, which makes it a good control layer for local coding workflows.
Step 4 — Install Ollama natively on macOS
Install Ollama using the macOS application. Ollama’s macOS docs describe the preferred method as dragging the app into Applications; Ollama then checks that the CLI is in your PATH. This native installation path is the one that can take advantage of Apple Metal acceleration.
Step 5 — Pull Gemma 4 models
Start with one main model and one fast helper model. This gives you a strong default without overcomplicating the editor configuration.
ollama pull gemma4:31b
ollama pull gemma4:e4b
ollama list

Use gemma4:31b as the main model for code generation, code review, planning, and harder reasoning tasks. Use gemma4:e4b for faster drafts, triage, and quick iterations.
Step 6 — Verify Ollama locally
Ollama exposes its API on localhost:11434 once it is running. Test that before you touch Continue.
curl http://localhost:11434/api/chat -d '{
  "model": "gemma4:31b",
  "messages": [
    {"role": "user", "content": "Reply with: Ollama and Gemma 4 are working."}
  ]
}'

Step 7 — Configure Continue
Create or edit your local Continue YAML configuration. Continue documents that model capabilities can be auto-detected, but you can explicitly set roles and capabilities when you want predictable behavior. It also notes that if you see memory errors, lowering contextLength is one of the first fixes.
name: Local AI Factory
version: 0.0.1
schema: v1

models:
  - name: Gemma 4 31B
    provider: ollama
    model: gemma4:31b
    apiBase: http://localhost:11434
    roles:
      - chat
      - edit
      - apply
    capabilities:
      - tool_use
      - image_input
    defaultCompletionOptions:
      contextLength: 8192
      temperature: 0.2
      reasoning: true
      keepAlive: 1800

  - name: Gemma 4 E4B Fast
    provider: ollama
    model: gemma4:e4b
    apiBase: http://localhost:11434
    roles:
      - chat
    capabilities:
      - tool_use
      - image_input
    defaultCompletionOptions:
      contextLength: 4096
      temperature: 0.2
      reasoning: false
      keepAlive: 1800

rules:
  - uses: file://./continue/rules/local-dev.md

Step 8 — Add Docker only where it helps
For macOS, the cleanest starting point is to let Docker run optional side services that talk to your host Ollama over the special DNS name host.docker.internal. Docker documents that this hostname resolves to the host machine on Docker Desktop.
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: ai-factory-open-webui
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://host.docker.internal:11434
      - WEBUI_AUTH=false
    volumes:
      - open-webui:/app/backend/data

volumes:
  open-webui:

This optional container does not replace Continue. It gives you a second interface for quick experiments, demos, and prompt testing while VS Code remains your coding environment.
6. If You Insist on Running Ollama in Docker on macOS
You can run the official Ollama container, but this is a compromise on your Mac. Ollama’s Docker docs provide the standard container command, while its FAQ clearly states that GPU acceleration is not available for Docker Desktop on macOS. So the containerized path is acceptable for tests, demos, or parity with a future Linux deployment, but not as your best-performance daily driver.
docker run -d   -v ollama:/root/.ollama   -p 11434:11434   --name ollama   ollama/ollama

If you use this path anyway, keep expectations realistic: you are trading performance for deployment uniformity.
7. Suggested Project Layout
ai-factory/
├── continue/
│   ├── config.yaml
│   └── rules/
│       └── local-dev.md
├── docker/
│   └── docker-compose.yml
├── prompts/
│   ├── architecture.md
│   ├── refactor.md
│   └── debug.md
├── workspaces/
│   ├── app-one/
│   └── app-two/
├── notes/
│   └── setup-log.md
└── desktop-shell/
    └── (future Tauri or Electron app)

This layout keeps the editor config, prompts, Docker assets, and future desktop shell cleanly separated. It also makes it easy to version-control the whole environment.
8. Performance Tuning Notes
• Do not start with the full 256K context in Continue just because Gemma 4 supports it. Begin with 4K–8K for stability and latency; grow only when a workflow needs it.
• Use the 31B model for planning, architecture, and harder code tasks. Use the smaller helper model when the request is short or repetitive.
• Keep Docker resource limits modest so containers do not crowd out Ollama’s memory.
• If Continue throws memory-related errors, lower contextLength first.
• Preload the model before a heavy work session by sending a light request to the Ollama API.
9. Path to Your Own Desktop App
Do not try to replace VS Code first. Treat this setup as your validated backplane. Once your workflows are stable, build a thin desktop shell around them.
Phase	Goal	What to build
1	Get productive now	VS Code + Continue + native Ollama + Gemma 4
2	Add orchestration	A Tauri or Electron shell for prompt libraries, model routing, run history, and job launching
3	Package workflows	Containerize helper APIs, RAG services, or team-facing local tools while leaving inference native on Mac

Recommendation summary: use Continue as your immediate IDE layer and treat your future desktop app as a control plane, not as a replacement editor. That sequence gets you working software sooner and preserves flexibility.
10. Official References
Continue Docs — Install: https://docs.continue.dev/ide-extensions/install
Continue Docs — Using Ollama with Continue: https://docs.continue.dev/guides/ollama-guide
Continue Docs — config.yaml reference: https://docs.continue.dev/reference
Google AI for Developers — Gemma 4 model card: https://ai.google.dev/gemma/docs/core/model_card_4
Google AI for Developers — Gemma thinking mode: https://ai.google.dev/gemma/docs/capabilities/thinking
Ollama — Gemma 4 library page: https://ollama.com/library/gemma4
Ollama Docs — macOS install: https://docs.ollama.com/macos
Ollama Docs — API introduction: https://docs.ollama.com/api/introduction
Ollama Docs — FAQ (Docker GPU caveat on macOS): https://docs.ollama.com/faq
Docker Docs — Install Docker Desktop on Mac: https://docs.docker.com/desktop/setup/install/mac-install/
Docker Docs — Networking how-tos (host.docker.internal): https://docs.docker.com/desktop/features/networking/networking-how-tos/
