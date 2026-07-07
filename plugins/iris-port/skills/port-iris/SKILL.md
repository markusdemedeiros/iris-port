---
name: port-iris
description: Ports Rocq Iris code to Lean. Use when the user asks to port a piece of Iris code to Lean, or when writing Lean code derived from Iris source code.
---

Port the given code from Rocq to Lean, following the porting rules, workflow, and style conventions in these files:

@${CLAUDE_PLUGIN_ROOT}/notes/RULES.md
@${CLAUDE_PLUGIN_ROOT}/notes/WORKFLOW.md
@${CLAUDE_PLUGIN_ROOT}/notes/STYLE_EXTENSIONS.md

When you hit iprop or proof-mode elaboration errors, consult the checklist in `${CLAUDE_PLUGIN_ROOT}/notes/IPROP_QUIRKS.md`.

When porting Rocq code, you should be using the following subagents:
- iris-rocq-expert: Query the Iris source code and the proof state at any given point.
- lean-style-critic: Make changes to the naming and style conventions of the code to ensure it is as clean as the Rocq implementation.
- lean4-fixer: Fix build errors when stuck, and improve the robustness of challenging code segments.

Ensure that you maintain a link between your code and the Rocq implementation, using the deprecation notices (`rocq_alias`) as described in `${CLAUDE_PLUGIN_ROOT}/notes/RULES.md`.
Keep your explanations brief and to the point.
The user is an Iris expert, and is eager to help you if stuck.
