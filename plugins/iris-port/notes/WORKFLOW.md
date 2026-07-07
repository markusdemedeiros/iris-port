# Porting Workflow

## Porting order
1. Read the Rocq source yourself. For short files (<300 lines), this is faster than the Rocq expert.
2. Grep/Glob for required Lean building blocks. Use direct tool calls, not the Explore agent.
3. Identify gaps: missing infrastructure, instance resolution issues, needed helpers. Solve these yourself.
4. Write the complete file. Sorry out only genuinely hard proofs.
5. Send the file to lean4-fixer to verify compilation and fix sorry'd leaves.
6. Apply all style rules yourself (aliases, naming, term-mode). Then send to lean-style-critic to apply fixes directly for what you missed.
7. Send to lean4-fixer again to verify the critic's edits still compile.
8. Verify 1:1 correspondence against the Rocq source.

## Scope
- Port everything unless told otherwise.
- Flag omissions upfront, not after the fact.
- If blocked, `sorry` out and note what's needed.

## When to use each subagent

**Iris-rocq-expert:**
- USE for: specific proof states at specific lines in complex proofs, understanding non-obvious typeclass chains.
- SKIP for: reading short files, listing declarations, anything you can get by reading the source directly.
- **Read the Rocq source yourself first.** Only call the expert when a proof is genuinely opaque — most proofs in Iris are short and readable. Don't ask for "proof state after every line" — ask targeted questions.

**lean4-fixer:**
- USE for: verifying compilation of near-complete files, fixing sorry'd leaf lemmas one at a time.
- SKIP for: designing proof architecture, discovering missing helpers, boilerplate that follows obvious patterns.
- The fixer is a verifier, not a developer. Send it 90%-correct code.
- **Keep prompts minimal.** Give it: (1) the Lean file path, (2) the Rocq source path, (3) pointers to `${CLAUDE_PLUGIN_ROOT}/notes/RULES.md`, `${CLAUDE_PLUGIN_ROOT}/notes/STYLE_EXTENSIONS.md`, and `${CLAUDE_PLUGIN_ROOT}/notes/IPROP_QUIRKS.md` (a checklist of recurring iprop/proof-mode elaboration fixes). Do NOT paste lemma lists, notation guides, or type descriptions — the fixer can read the file and discover these itself. The Iris Rocq source is a high-quality reference proof script; tell the fixer it can use the `iris-rocq-expert` subagent to inspect proof states if it gets stuck understanding the original proof strategy.

**lean-style-critic:**
- USE for: catching style issues you missed after applying all known rules yourself.
- SKIP for: applying rules you already know (aliases, naming). Do that yourself first.
- **Keep prompts minimal.** Give it: (1) the Lean file path, (2) the Rocq source path, (3) pointers to `${CLAUDE_PLUGIN_ROOT}/notes/RULES.md` and `${CLAUDE_PLUGIN_ROOT}/notes/STYLE_EXTENSIONS.md`. The Iris Rocq source is a high-quality reference proof script; tell the critic it can use the `iris-rocq-expert` subagent to query the original proofs for naming and 1:1 correspondence checks.
- **Let it edit directly.** Tell the critic to apply its fixes to the file, not just report them. Follow up with lean4-fixer to verify the edits compile.

**Explore agent:**
- USE for: genuinely open-ended codebase questions requiring many searches.
- SKIP for: finding specific files. Use Glob/Grep directly.

## General rules
- Do not repeat notes file contents in agent prompts — point to the files.
- Do not duplicate work a subagent is doing.
- Wait for agent results before making further edits.
- Evaluate agent suggestions critically — do not accept blindly.
- **Think first, delegate second.** Do the hard design work (instance resolution, proof structure, missing infrastructure) yourself. Subagents verify and polish; they don't architect.
- **Don't re-research what you already know.** Trust your context.
- **Mechanical code doesn't need subagents to write.** Type abbreviations, `inferInstance`, aliases, and obvious pattern-following code should be written directly, then verified.
