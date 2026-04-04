# Desktop Shell

This directory is reserved for the future **Tauri or Electron** desktop control plane.

## Purpose (Phase 2 and 3)
Once your VS Code + Continue + Ollama workflows are proven and stable, build a thin desktop shell around them. This shell is **not** a replacement for VS Code — it is a control plane for:
- Prompt library management
- Model routing (choosing 31B vs E4B dynamically)
- Run history and session logs
- Launching background jobs
- Exposing local tools to non-developer users

## Recommended Technology
- **Tauri** — preferred for smaller binary size and native Rust backend
- **Electron** — acceptable if you want to reuse more JavaScript tooling

## Do Not Build This First
Return to this directory only after:
1. Native Ollama + Gemma 4 are working and stable
2. Continue workflows are proven across real projects
3. You have identified specific pain points that a desktop shell would solve

## References
- Tauri: https://tauri.app
- Electron: https://www.electronjs.org
