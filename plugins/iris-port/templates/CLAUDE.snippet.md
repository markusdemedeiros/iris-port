# Optional: wire the porting rules into every session

Plugins load skills and agents on demand, but they do **not** automatically inject the
notes files into your session context the way a project `CLAUDE.md` does. The original
Iris-Lean setup relied on `CLAUDE.md` loading the rules into *every* turn.

If you want that always-on behavior, add a block like this to your project's `CLAUDE.md`.
Adjust the path to wherever the plugin is installed (Claude Code exposes it via
`${CLAUDE_PLUGIN_ROOT}` inside plugin files, but a project `CLAUDE.md` needs a concrete
path — copy the notes into your repo, or point at the installed plugin directory):

```markdown
# Iris-Lean porting effort
Refer to notes/RULES.md for the porting rules.
Refer to notes/WORKFLOW.md for the porting workflow and agent-usage patterns.
Refer to notes/STYLE_EXTENSIONS.md for extended style rules.
Refer to notes/HOUSE_STYLE.md for the accumulated house style.
```

If you'd rather keep the notes only inside the plugin, skip this — the `/port-iris`
skill already points the agents at `${CLAUDE_PLUGIN_ROOT}/notes/*` when it runs.
