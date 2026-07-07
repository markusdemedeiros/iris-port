# Extended Style Rules

These extend the base rules in RULES.md.

## 1. No Rocq docstrings
Do not add `-- Rocq: foo` comments. Use `rocq_alias` for name divergences; matching names need no annotation.

## 2. Short theorems on one line
If signature and term-mode proof fit on one line, keep them together.

## 3. Multiple rewrites in a single `rw`
`rw [foo, bar]` is preferred over separate `rw` calls.

## 4. Author field
Leave the `Authors:` field blank in copyright headers.

## 5. Open namespaces to shorten proofs
Open relevant namespaces to eliminate verbose qualified names. After opening, remove all newly-redundant qualifiers. A qualified name should only remain when the unqualified form would shadow or cause ambiguity.

## 6. Eliminate `haveI`
Restructure proofs to avoid `haveI := ha` — use instance arguments, `@`, or reorganize.

## 7. `rocq_alias` on every declaration
Every declaration (def, theorem, instance, abbrev) must have a `rocq_alias` directly below it with the exact Rocq name, so that every Rocq name is searchable at the root level:
```lean
theorem agreeN ... := ...
rocq_alias excl_auth_agreeN := ExclAuth.agreeN
```
Port files must `import Iris.Std.RocqAlias`.

## 8. Delete inferrable implicit arguments
Prefer `theorem foo : ...` over `theorem foo {a b : A} : ...` when `a` and `b` can be inferred. We differ from mathlib here.

## 9. Term-mode vs tactic proofs
Prefer term-mode for clean one-liners. Use tactic mode when destructuring or rewriting is needed. Do not force term-mode if it duplicates subexpressions — use `let` to bind shared terms.

## 10. Drop `inferInstance` instances
Do not port Rocq instances whose Lean proof would be just `inferInstance`. The typeclass system handles them automatically, so naming them adds noise. Do not create a `rocq_alias` for them either.

