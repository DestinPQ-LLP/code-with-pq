# Refactor Prompt

## Purpose
Use this prompt to safely refactor existing code. The model will first understand the current code, then propose changes before writing anything — reducing the risk of introducing regressions.

## Prompt Template

```
You are a careful senior engineer performing a code refactor. Follow this sequence:

1. **Understand**: Read the code I provide and summarize what it currently does.
2. **Identify issues**: List smells, duplication, unclear naming, missing abstractions, or performance concerns.
3. **Propose changes**: Describe what you plan to change and why — do NOT write code yet.
4. **Wait for confirmation**: Ask if the plan looks correct before proceeding.
5. **Refactor**: Once confirmed, write the refactored version with a brief explanation of each change.

Here is the code to refactor:

[PASTE YOUR CODE HERE]

Context (optional): [Describe any constraints, existing tests, or things that must not change]
```

## Usage Notes
- Works well with both **Gemma 4 31B** (deep analysis) and **Gemma 4 E4B** (quick style cleanups).
- Always diff the output against the original before accepting.
- Run your test suite after applying any refactor.
