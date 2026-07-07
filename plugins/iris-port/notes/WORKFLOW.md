# Porting Workflow

## Porting order
1. Read `tactics.md` (and `proofmode.md`) at the root of the iris-lean checkout to refresh your knowledge of the current iris-lean proof-mode tactics — their names, syntax, and semantics drift over time.
2. Read the Rocq source yourself. For short files (<300 lines), this is faster than the Rocq expert.
3. Grep/Glob for required Lean building blocks. Use direct tool calls, not the Explore agent.
4. Identify gaps: missing infrastructure, instance resolution issues, needed helpers. Solve these yourself.
5. Write the complete file. Sorry out only genuinely hard proofs.
6. Send the file to lean4-fixer to verify compilation and fix sorry'd leaves.
7. Apply all style rules yourself (aliases, naming, term-mode). Then send to lean-style-critic to apply fixes directly for what you missed.
8. Send to lean4-fixer again to verify the critic's edits still compile.
9. Verify 1:1 correspondence against the Rocq source.
10. If the port surfaced any improvements to the `/port-iris` plugin itself (new rules, notes, agent or workflow tweaks), ask the user whether they want to contribute them upstream to https://github.com/markusdemedeiros/iris-port. If yes, generate a self-contained `make_pr.sh` for the user to run on their **own** GitHub-authenticated machine — the container has no `gh` and no push credentials, and its filesystem is **not** shared with the user, so the container can't push and can't hand them a file directly. Build the script like this:
    - **Locate the two copies.** You edit the *live* plugin in the cache at `~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/`; the *git repo* is the marketplace checkout at `~/.claude/plugins/marketplaces/<marketplace>/`, where the plugin sits under `plugins/<plugin>/…`. (`diff -rq` the two to see exactly which files you changed; `git -C <checkout> log -1` gives the base commit.)
    - **Make the patch, authored by Claude.** Clone **fresh from the remote** (`git clone <url>`) so you branch off *current* `main` — the local marketplace checkout may be stale (e.g. a prior PR already merged), and basing the patch on it makes `git am` re-apply merged hunks and conflict. Branch, copy your edited files from the cache into `plugins/<plugin>/…`, then commit *as Claude*: `git -c user.name="Claude" -c user.email="noreply@anthropic.com" commit -m "…"` (do **not** author it as the user, and skip the `Co-Authored-By` trailer since Claude is the author). Then `git format-patch -1`.
    - **Write `make_pr.sh`** to the project root. It must, in order: clone the repo over **SSH** (`git@github.com:<owner>/<repo>.git`); `git checkout -b <branch>`; set a committer-identity fallback (`git config user.email >/dev/null 2>&1 || git config user.email …`, same for `user.name`) so `git am` can't fail on a missing identity; decode an **embedded base64** copy of the patch (base64 is transport-safe — raw patches corrupt on paste because of Unicode like `∗`/`⊢`/`§` and exact whitespace) via `openssl base64 -d` and `git am` it; `git push -u origin <branch>`; then `gh pr create --repo … --base main --head <branch> --title … --body …` if `gh` exists, else print `https://github.com/<owner>/<repo>/compare/main...<branch>?expand=1`.
    - **Verify before handing off:** `bash -n make_pr.sh` (syntax); the embedded base64 decodes byte-identical to the patch (`… | openssl base64 -d | cmp - patch`); and the patch `git am`s cleanly onto a fresh clone of `main` (set a temp identity for the test). Then tell the user the path and to run `bash make_pr.sh`.

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
