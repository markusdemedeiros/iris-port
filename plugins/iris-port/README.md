# iris-port

Claude Code plugin for porting Rocq (Coq) [Iris](https://iris-project.org/) code to Lean 4,
mathlib-style. Bundles the `/port-iris` orchestrator, three subagents, the rules/style
knowledge base, and a `/learn-diff` style-training loop.

## Layout

```
skills/port-iris      /port-iris — orchestrator (entry point)
skills/learn-diff     /learn-diff — trains the style critic from a reviewed diff
agents/               iris-rocq-expert · lean4-fixer · lean-style-critic
notes/                RULES · WORKFLOW · STYLE_EXTENSIONS · HOUSE_STYLE · IPROP_QUIRKS
diffs/                empty — drop reviewed diffs here for /learn-diff
.mcp.json             lean-lsp + rocq-mcp
```

## Setup

- **lean-lsp** (required) — pre-configured to run `uvx lean-lsp-mcp` (needs
  [`uv`](https://docs.astral.sh/uv/)). Needs a Lean 4 + Lake toolchain.
- **rocq-mcp** (optional, not shipped) — enables live Rocq proof-state inspection.
  Without it, `iris-rocq-expert` degrades to reading/grepping the Rocq source as text
  (and fetching source files from GitHub if none is checked out locally); it will not
  install or compile Rocq. To opt in, add a block like this to `.mcp.json` with your
  server's launch command and set `ROCQ_PROJECT_ROOT` to your Iris Rocq checkout:

  ```json
  "rocq-mcp": {
    "type": "stdio",
    "command": "<your-rocq-mcp-command>",
    "args": [],
    "env": { "ROCQ_PROJECT_ROOT": "${CLAUDE_PROJECT_DIR}" }
  }
  ```

Install, then run `/port-iris`. Plugins load notes on demand, not into every turn — see
`templates/CLAUDE.snippet.md` for optional always-on wiring.

## Workflow

`/port-iris` reads the notes, consults **iris-rocq-expert** for opaque Rocq proof states,
writes the Lean port (`sorry` on hard proofs), then **lean4-fixer** compiles/fills leaves
and **lean-style-critic** applies conventions. `/learn-diff` feeds human cleanups back
into the critic's memory.

Learned state (agent memories, `diffs/`) ships empty and grows with use;
`notes/HOUSE_STYLE.md` is a reusable seed.
