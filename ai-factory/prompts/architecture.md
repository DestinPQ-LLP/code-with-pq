# Architecture Prompt

## Purpose
Use this prompt when planning a new system, feature, or significant refactor. Feed it context about what you are building and let Gemma 4 31B reason through trade-offs, layer boundaries, and sequencing before any code is written.

## Prompt Template

```
You are a senior software architect. I will describe a system I am designing. Your job is to:
1. Identify the major components and their responsibilities.
2. Define the boundaries and interfaces between components.
3. Flag risks, anti-patterns, and assumptions that need validation.
4. Suggest a phased build order that delivers value early and avoids deep coupling.
5. Ask clarifying questions if any requirement is ambiguous.

Here is the system I am designing:

[PASTE YOUR SYSTEM DESCRIPTION HERE]
```

## Usage Notes
- Use this with **Gemma 4 31B** for maximum reasoning depth.
- Start broad, then narrow down with follow-up prompts.
- Save the output to `notes/` before you start coding.
