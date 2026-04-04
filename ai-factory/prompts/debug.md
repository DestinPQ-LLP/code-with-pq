# Debug Prompt

## Purpose
Use this prompt when you have a bug, an unexpected output, or a failing test. The model will reason through the problem systematically rather than guessing at fixes.

## Prompt Template

```
You are an expert debugger. I have a problem I cannot resolve. Work through it with me using this approach:

1. **Restate the problem**: Describe in your own words what is happening versus what should happen.
2. **Form hypotheses**: List the most likely causes, ordered by probability.
3. **Suggest diagnostics**: For each hypothesis, describe a targeted check or log statement to confirm or rule it out.
4. **Do not jump to a fix**: Only propose a fix after we have narrowed down the root cause through evidence.

Here is my situation:

**What I expected to happen:**
[DESCRIBE EXPECTED BEHAVIOR]

**What actually happened:**
[DESCRIBE ACTUAL BEHAVIOR, including any error messages or stack traces]

**Relevant code:**
[PASTE CODE OR LINK TO FILE]

**What I have already tried:**
[LIST ANYTHING YOU HAVE TRIED]
```

## Usage Notes
- Use **Gemma 4 31B** for complex multi-layer bugs (async issues, memory problems, race conditions).
- Use **Gemma 4 E4B** for simple typos, missing imports, or quick syntax issues.
- Always include error output and stack traces — they are the most useful context.
