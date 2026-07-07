# iris-lean plugin marketplace

A Claude Code plugin marketplace for the Iris-Lean porting effort.

Currently ships one plugin:

- **iris-port** — a protocol for faithfully porting Rocq (Coq) [Iris](https://iris-project.org/)
  code to Lean 4, mathlib-style. See [`plugins/iris-port/README.md`](plugins/iris-port/README.md).

## Install

```
/plugin marketplace add markusdemedeiros/iris-port
/plugin install iris-port@iris-lean
```

The first command registers this repository as a marketplace; the second installs the
`iris-port` plugin from it.

Third-party marketplaces don't auto-update by default. To keep `iris-port` current, enable
auto-update — either toggle it in `/plugin` → **Marketplaces** → `iris-lean` → **Enable
auto-update**, or add the `autoUpdate` flag to `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "iris-lean": {
      "source": { "source": "github", "repo": "markusdemedeiros/iris-port" },
      "autoUpdate": true
    }
  }
}
```

## Repository layout

```
.claude-plugin/marketplace.json   # marketplace manifest (lists the plugins below)
plugins/
└── iris-port/                     # the plugin
    ├── .claude-plugin/plugin.json
    ├── .mcp.json                  # lean-lsp (rocq-mcp is opt-in; see plugin README)
    ├── skills/  agents/  notes/
    ├── diffs/  templates/
    └── README.md
```

To add more plugins later, drop them under `plugins/<name>/` and add an entry to
`.claude-plugin/marketplace.json` with `"source": "./plugins/<name>"`.
