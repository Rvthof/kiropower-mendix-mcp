---
inclusion: always
---

# Writing Files

## Critical Rule: Large Files Must Use fsWrite + fsAppend

`fsWrite` truncates content beyond ~50 lines. Any file longer than 50 lines — including scripts, requirements docs, and design documents — **must** be split across multiple calls.

### Required Pattern

```
1. fsWrite  → first ~40 lines
2. fsAppend → next ~40 lines
3. fsAppend → next ~40 lines
... repeat until file is complete
```

Never put more than ~40–45 lines in a single `fsWrite` or `fsAppend` call.

### Example: Writing a 120-line file

```
fsWrite  path/to/file.md  ← lines 1–40
fsAppend path/to/file.md  ← lines 41–80
fsAppend path/to/file.md  ← lines 81–120
```

## Never Retry by Creating a New File

If a file has an issue, **fix the existing file** using `strReplace` or by rewriting it with `fsWrite` + `fsAppend`. Do NOT create `file-part2.md`, `file-fix.md`, numbered variants, etc.

One logical document = one file.

## File Size Guidance

| File content | Approximate lines | Chunks needed |
|---|---|---|
| Short doc / small script | < 80 lines | fsWrite + 1 fsAppend |
| Medium doc / several sections | 80–200 lines | fsWrite + 3–4 fsAppend |
| Large doc / full feature spec | 200–500 lines | fsWrite + 5–10 fsAppend |

When in doubt, use smaller chunks. An extra `fsAppend` call costs nothing; a truncated file wastes a full retry cycle.
