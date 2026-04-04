# Local Development Rules

## General Philosophy
- Prioritize correctness over brevity. This is a local AI factory, not a chatbot.
- Always prefer idiomatic, readable code over clever one-liners.
- Prefer explicit over implicit. Ambiguous requirements should trigger a clarifying question.

## Code Style
- Follow the language-native conventions for the project being worked on.
- Use meaningful variable and function names.
- Add comments only where the intent is non-obvious — avoid noise comments.
- Prefer small, focused functions and components.

## Planning and Architecture
- When asked to build something non-trivial, outline a brief plan before writing code.
- Flag potential design issues early rather than building first and refactoring later.
- Prefer reversible decisions. Avoid coupling that is hard to undo.

## Context Management
- This setup uses Gemma 4 31B for deep reasoning and code generation.
- Use Gemma 4 E4B for lightweight tasks: quick lookups, short drafts, and triage responses.
- Do not grow context length beyond 8192 unless a specific workflow demands it.

## Local-First Principles
- All model inference runs natively via Ollama on the host machine (Apple Metal acceleration).
- Docker is reserved for helper services only — not for primary LLM inference on macOS.
- Avoid making unnecessary outbound calls during development.

## Performance Awareness
- Before a heavy work session, preload the model with a lightweight request.
- If memory errors occur, lower `contextLength` in config.yaml before anything else.
- Use the 31B model for hard problems; use the smaller model for fast iterations.

## Security and Privacy
- Never commit secrets, tokens, or credentials to the repository.
- Treat all local data as potentially sensitive — do not log or expose prompt content unnecessarily.
