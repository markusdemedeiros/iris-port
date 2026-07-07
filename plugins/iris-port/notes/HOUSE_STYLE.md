---
name: House style rules (consolidated)
description: Complete house style rules for iris-lean, organized by category — naming, implicits, scoping, class design, proof style, formatting, documentation. Derived from manual cleanup diffs across PRs #45, #48, #50, #117, #148, #160.
type: feedback
---

# House Style Rules

## Naming

1. **Mathlib casing convention.** Types/Props/classes: `UpperCamelCase` (`structure Foo`, `class Bar`). Theorems (terms of `Prop`): `snake_case` (`op_congr_left`, `later_true`). Other terms of `Type` (functions, defs): `lowerCamelCase` (`toFun`, `transpAp`). When an `UpperCamelCase` name appears inside a `snake_case` name, use `lowerCamelCase` (`map_natCast` not `map_NatCast`).

2. **Mathlib naming conventions.** `_left`/`_right` not `_l`/`_r`. `map_` prefix for morphism fields (`f_ne` → `map_ne`, `homomorphism` → `map_op`). Rearrangement lemmas: `op_op_op_comm`, `op_left_comm`.

3. **Suffix conventions.** `_equiv` for `≡`, `_dist` for `≡{n}≡`, `_of_forall_equiv` or `_pointwise` for pointwise variants.

4. **Theorems named per mathlib conventions.** The theorem name should describe what it proves, with the primary subject as a namespace or prefix.

5. **Short constructor names — drop type prefix.** `Excl.exclInvalid` → `Excl.invalid`. The namespace provides context.

7. **Names reflect the abstraction.** `HeapOF` → `PartialMapOF` when it works for any `PartialMap`.

8. **Unambiguous theorem names.** `singleton_map_ne` → `singleton_map_none` (it returns `none`, not "not equal").

9. **Greek letters for type variables.** `α`, `β`, `γ` not `A`, `B`, `C`. Domain-specific names (`PROP`, `M`, `K`, `V`) are fine.

10. **Type names in theorem names are lowercase.** `later_True` → `later_true`. Theorems are fully snake_case.

## Implicit Arguments

11. **Implicit binders in class fields** when arguments are inferrable. `op_assoc : ∀ {a b c}, ...` not `∀ a b c, ...`.

12. **Implicit binders in hypothesis parameters.** `(h : ∀ {i x}, l[i]? = some x → ...)` not `(h : ∀ i x, ...)`.

13. **Function arguments implicit when inferrable.** `{Φ Ψ : K → V → M}` not `(Φ Ψ : K → V → M)` when determined by other args.

14. **Don't re-bind variables already in scope.** If `{p : Bool}` is in a `variable` block, don't repeat `{p}` in theorem signatures.

15. **Remove inferrable named arguments from type ascriptions.** When `ExclAuthR (F := F) (A := A)` has inferrable named args, drop them: `ExclAuthR (F := F)`. Extends rule 13 to ascriptions in expression bodies.

16. **Narrow type ascription scope.** Ascribe the minimal subexpression needed for inference. `(●E a : ExclAuthR (F := F)) • ◯E b` not `((●E a : ExclAuthR (F := F)) • ◯E b : ...)`.

## Variable & Scope Management

15. **Consolidate `variable` declarations.** Merge adjacent `variable` lines into one when they share scope.

16. **Merge `open` statements.** `open OFE Iris.Std` not separate lines.

17. **Open namespaces to eliminate qualified names.** Then remove all now-redundant qualifiers.

18. **`section` vs `namespace`.** Use `namespace` when definitions should be namespaced (`Foo.bar`). Use `section` only for scoping variables without a namespace.

19. **Sections to scope variables.** `section Hom` / `end Hom` for variable blocks that apply to a subset of theorems.

20. **Notation inside namespaces.** Don't leave notation floating between `end Foo` and a new section.

21. **Never use `omit`.** Restructure variable scoping so `omit` is unnecessary.

22. **Explicit function parameters over `variable (x)`.** `variable` is for shared context; actual function arguments belong in signatures.

## Class & Instance Design

23. **Eliminate `haveI`/`letI` for inferrable instances.** Use `attribute [instance]`, `variable`, or restructure so typeclass resolution handles it.

24. **Instance arguments for structures.** `[H : MonoidHomomorphism ...]` enables dot notation: `H.map_unit`, `H.map_op`.

25. **Element-level predicates as `class`, not `def`.** `class DiscreteE ... : Prop where discrete : ...` enables `[DiscreteE x]` and `.discrete`.

26. **`instance ... where` for typeclass derivations.** Not `theorem ... : Foo := ...`.

27. **`where` syntax for simple instances.** Not `⟨...⟩` or `by refine { ... }`.

28. **Inline CMRA/typeclass field definitions.** Don't define separate `@[simp] def Foo.pcore` just to plug into an instance. Exception: `Valid`, `Equiv`, `Dist` that need `@[simp]` independently.

29. **Drop redundant binders in instance fields.** `assoc := by simp` not `assoc {x y z} := by simp`.

30. **Extract base classes — don't duplicate.** Shared definitions belong in the base class. Remove duplicates from subclasses.

31. **Remove duplicate notation/instances** inherited from parent classes.

## Proof Style

32. **Term-mode over tactic-mode** when branches are short. `match l with | .nil => .rfl | .cons _ _ => ...`.

33. **Dot notation.** `.rfl`, `.symm`, `.trans`, `.nil`, `.cons`. Also `H.dist` over `equiv_dist.mp H _`, `(IH H).le` over `Dist.le (IH ...) ...`.

34. **Pipe chains.** `op_congr_right (..) |>.trans op_assoc.symm`.

35. **`simpa` and `grind`** over multi-step simp+closer patterns.

36. **Prefer `suffices` over `rw [show ... from ...]`.** Use `suffices H : goal by rwa [...]` instead of `rw [show X = Y from proof, ...]`. Separates the rewrite from the proof of the simplified statement.

36. **`[DecidableEq K]` over `open Classical in`** when branching on equality.

37. **Remove `have` bindings — prefer backwards reasoning.** Inline single-use `have`s. Use `apply`/`refine` chains over forward `have` chains.

38. **No unnecessary parentheses in tactic arguments.** `rcases get? a x` not `rcases (get? a x)`.

39. **Remove redundant outer parentheses in theorem signatures.** `✓{n} (●E a : T) • ◯E a` not `(✓{n} ((●E a : T) • ◯E a))` when precedence makes grouping clear.

39. **`refine .trans ?_ rhs`** over `apply flip Dist.trans rhs`.

40. **Compress `intro`/`apply`/`intro`.** `refine fun n => f fun k => ?_`.

41. **`exact` not `apply` when no goals remain.**

42. **Prefer `..` for inferrable arguments.** `exact (foo ..)` not `exact (foo f g _ _)`.

43. **`next` over `rename_i`.** `next h => ...` not `· rename_i h; ...`.

44. **Structured `induction ... with`** over bare `induction` + `rename_i`.

45. **`rintro` to combine intro and case split.** `rintro (_|i)` not `intro i; cases i`.

46. **`rintro ⟨a, rfl⟩`** to substitute equalities immediately.

47. **`·.casesOn` over `Bool.rec`.** `p.casesOn .rfl foo` not `Bool.rec .rfl foo p`.

48. **`.symm` on arguments over `_mpr` lemmas.** `foo_mp h.symm` not separate `foo_mpr`.

49. **Collapse identical branches with `<;>`.** DRY within proofs.

50. **`obtain` over `rcases` for simple destructuring.** `obtain _|x' := expr` not `rcases expr with _|x'`.

51. **`Option.map` over `<$>`** in algebraic contexts. `(pcore x).map f` not `f <$> pcore x`.

52. **Convert trivial tactic proofs to term.** `by exact H` → `H`. `by rfl` → `rfl`.

53. **`have` not `let` for proof bindings in tactic mode.** In term-mode, `let` is fine for binding shared subexpressions (see STYLE_EXTENSIONS §9).

54. **Hoist shared computations out of case splits.** Same `have` in multiple branches → move before `cases`.

55. **`@[elab_as_elim]`** on custom induction principles.

56. **No `@[simp]` on predicates/type definitions** that shouldn't auto-unfold.

57. **Delete trivial extensionality/congruence lemmas** that just wrap `congrArg`/`funext`.

58. **Unicode `→` over ASCII `->`.** Always.

59. **Named hypotheses before the colon.** `theorem foo (h : H) : P` not `theorem foo : H → P := fun h =>`. This applies to explicit (especially `Prop`-valued) hypotheses — use judgment. Inferrable implicit arguments should be deleted per STYLE_EXTENSIONS §8.

60. **`:=` at end of signature line.** Proof body indented below.

## Formatting

61. **No double blank lines.** At most one blank line between definitions.

62. **No stray whitespace.** No double spaces within code, no trailing whitespace.

63. **Space before `:=`.** `def foo : T :=` not `def foo : T:=`.

64. **No leading space after `⟨`.** `⟨foo, bar⟩` not `⟨ foo, bar⟩`.

65. **Line width ~100 chars.** Wrap signatures with 4-space continuation indent.

66. **Declaration ordering.** Within a namespace: definitions, notation/syntax, law classes, theorems.

## Documentation

67. **No "Corresponds to Rocq's ..." in docstrings.** Describe behavior in Lean terms. Use `rocq_alias` for Rocq name mapping.

68. **No `abbrev` aliases for Rocq names.** Use `rocq_alias` if available; otherwise leave a comment. The macro is still in development.

69. **Delete commented-out code.** Version control preserves history.

70. **Remove stale TODO/FIXME comments.**

71. **Remove `set_option` warning suppressions.** Fix the underlying issue.

72. **Remove stale comments** — abandoned design notes, restated code, empty section labels.

73. **Docstrings on notation-enabling instances.** Short docstring explaining what `∅`, `⊆`, `∪`, `\`, `∈` etc. they enable.

74. **Section headers use `/-! ## Title -/`** not plain `/- Title -/`.

75. **Wrap docstrings at ~100 chars.** Same line width as code.
