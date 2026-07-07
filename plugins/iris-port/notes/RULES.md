# Iris-lean porting effort
Your goal is to faithfully port the Rocq implementation of the Iris framework to Lean 4.
A team of experts (including the user) has begun this effort. They will be carefully reviewing your code. 
Your job is to produce high-quality Lean 4 code, that minimizes the amount of effort required by the human reviewers.

## Golden Rules (IMPORATNT)
- The FIRST GOLDEN RULE: You are never premitted to use git. git, or any version control command, is expressly forbidden under any circumstance. 
    + If a git command is necessary, you may ask the user to execute it for you. 
- The SECOND GOLDEN RULE: Do not spin on problems you are stuck on. The user is an expert. Sorry out and revisit difficult lemmas, and ask for assistance.

## Guidelines for using Iris (Rocq)
- Rocq Iris code is READ ONLY. Do not modify it. Treat it like an artifact for querying information out of, NOT something to modify yourself. 
- Use Rocq-MCP to query the proof state of Rocq code at different points.

## Guidelines for using Lean
- Ask for help when stuck. The user is an expert. Do not spin on a problem — ask.
- The file must compile. Every edit must leave the project in a buildable state (no errors, sorries are allowed temporarily).
- Do not attempt to write your own proofs for lemmas that are present in Rocq. Collaborate with the Iris Rocq subagent for information about intermediate states. 

## Guidelines for reviewing Lean code
- The ideal code matches the style of existing code in the reposiroty. 
- The ideal code conforms to mathlib naming conventions, and the mathlib style guide
    + https://leanprover-community.github.io/contribute/naming.html
    + https://leanprover-community.github.io/contribute/style.html
- The ideal code documents when Iris-Lean names diverge from Iris-Rocq. Use `rocq_alias` if available; otherwise leave a comment. The `rocq_alias` macro is still in development.
- The key rules of Lean code review:
    + One line equals one idea. Do not splice tactics together with semicolons to artifically.
    + Every tactic's outcome should be easily predictable. Prefer `refine` to `apply`. 
    + `have` statements should be minimized. Backwards reasoning is better than forwards.


