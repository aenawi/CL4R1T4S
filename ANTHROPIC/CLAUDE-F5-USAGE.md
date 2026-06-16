# Using the Fable 5 Prompts with Claude Code — Reference

This folder contains three related prompt files. This document explains what each is and the
**exact instructions** for using it with Claude Code.

## TL;DR — which file do I use?

| File | Size | Use with | What it is |
|------|------|----------|------------|
| `CLAUDE-F5.md` | ~122 KB | — (reference / experimental) | Original Fable 5 **chat** prompt (claude.ai). Carries the chat tool catalog, artifacts, web/image search. **Not** a coding-agent prompt. |
| `CLAUDE-F5-CLAUDE-CODE.md` | ~137 KB | `--system-prompt-file` | **Full replacement**: a reconstructed Claude Code agentic harness **+** the full Fable core preserved verbatim. One self-contained prompt. |
| `CLAUDE-F5-APPEND.md` | ~25 KB | `--append-system-prompt-file` | **Recommended.** Just the Fable behavioral/safety/tone core, layered on top of Claude Code's genuine built-in harness. |

**Recommended default:** `CLAUDE-F5-APPEND.md` with `--append-system-prompt-file`. It keeps Claude Code's
real, current harness (live tool schemas + the auto-injected environment block) and only adds Fable's
values/safety/tone, so nothing drifts out of date.

## Key facts about the system-prompt flags

- There are four flags: `--system-prompt`, `--system-prompt-file`, `--append-system-prompt`, `--append-system-prompt-file`.
- The `*-file` variants read the prompt from a file; the non-file variants take an inline string.
- `--system-prompt*` **replaces** Claude Code's built-in prompt; `--append-system-prompt*` **adds to** it.
- `--system-prompt` and `--system-prompt-file` are **mutually exclusive**; an append flag can be combined with a replacement flag.
- All four work in **both** interactive mode and print mode (`-p`).
- They are **per-invocation** — they apply only to that run and are not saved. See "Make it persistent" below.

```bash
# Optional: set once per shell so the examples below are copy-paste clean
DIR=/home/aldhaheri/Workspaces/Development/side-projects/CL4R1T4S/ANTHROPIC
```

---

## 1. `CLAUDE-F5.md` — the original Fable **chat** prompt

This is the claude.ai chat prompt. It is **not meant to drive Claude Code**: full-replacing with it
strips the agentic harness, and appending it injects a chat tool catalog (`image_search`, `weather_fetch`,
`recipe_display_v0`, a Python-sandbox `bash_tool`, etc.) that doesn't exist in the CLI. Use it for
reference, or to run the raw chat persona for experimentation only.

```bash
# Experimental only — model loses its coding-agent harness:
claude --system-prompt-file "$DIR/CLAUDE-F5.md"
```

For real Claude Code work, use file 2 or 3 instead.

---

## 2. `CLAUDE-F5-CLAUDE-CODE.md` — full replacement (self-contained)

Replaces Claude Code's built-in prompt entirely with the reconstructed harness + full Fable core.

```bash
# Interactive session:
claude --system-prompt-file "$DIR/CLAUDE-F5-CLAUDE-CODE.md"

# One-shot / scripted (print mode):
claude -p "refactor utils.py" --system-prompt-file "$DIR/CLAUDE-F5-CLAUDE-CODE.md"
```

When to use: you want one fully self-contained prompt with no reliance on the built-in harness.

---

## 3. `CLAUDE-F5-APPEND.md` — append on top of the built-in harness (recommended)

Keeps Claude Code's genuine, current harness (live tool schemas + auto-injected environment block) and
layers only Fable's values/safety/tone.

```bash
# Interactive session:
claude --append-system-prompt-file "$DIR/CLAUDE-F5-APPEND.md"

# One-shot / scripted (print mode):
claude -p "add tests for auth.ts" --append-system-prompt-file "$DIR/CLAUDE-F5-APPEND.md"
```

When to use: this is the safe default — nothing drifts out of date because the real harness stays intact.

---

## Make it persistent (so you don't retype the flag)

The flags don't persist, so wrap your choice in a shell alias or function in `~/.zshrc`:

```bash
# Pick ONE of these (file 3 / append shown — the recommended default):
alias claude-fable='claude --append-system-prompt-file "/home/aldhaheri/Workspaces/Development/side-projects/CL4R1T4S/ANTHROPIC/CLAUDE-F5-APPEND.md"'

# Function variant that forwards extra args (e.g. claude-fable -p "fix bug"):
claude-fable() {
  claude --append-system-prompt-file \
    "/home/aldhaheri/Workspaces/Development/side-projects/CL4R1T4S/ANTHROPIC/CLAUDE-F5-APPEND.md" "$@"
}
```

Then `source ~/.zshrc` and just run `claude-fable`. Swap the path to `CLAUDE-F5-CLAUDE-CODE.md` (and the
flag to `--system-prompt-file`) if you prefer the full-replacement variant.

There's no `settings.json` field for a raw system prompt. For a persistent persona that also works in the
Claude Agent SDK, the alternative is an **output style** (`~/.claude/output-styles/fable.md` with
`keep-coding-instructions: true` in its frontmatter), activated via `/output-style`.

---

## Quick sanity check that it loaded

The flags are silent, so confirm behaviorally — start a session with your chosen flag and ask something
the Fable layer governs:

```
> What model are you and what's your knowledge cutoff?
```

With the Fable layer active it should describe itself as **Claude Fable 5** with an **end-of-Jan-2026**
cutoff. If you used append (file 3), it should still behave as a coding agent (edit files, run tools);
if it stops acting agentic, you accidentally used a `--system-prompt`/`-file` replacement flag instead
of the append one.

---

## Claude Agent SDK equivalents

The same layering applies when scripting with the SDK:

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";
import { readFileSync } from "fs";

// Append (recommended): keep the claude_code preset, add the Fable core
query({
  prompt: "...",
  options: {
    systemPrompt: {
      type: "preset",
      preset: "claude_code",
      append: readFileSync("./CLAUDE-F5-APPEND.md", "utf8"),
    },
  },
});

// Full replacement: run only the self-contained combined prompt
query({
  prompt: "...",
  options: {
    systemPrompt: readFileSync("./CLAUDE-F5-CLAUDE-CODE.md", "utf8"),
  },
});
```

Note: the SDK's default (when `systemPrompt` is unset) is a minimal tool-calling prompt, **not** the
`claude_code` preset — set `type: "preset", preset: "claude_code"` to match the CLI's built-in behavior.
