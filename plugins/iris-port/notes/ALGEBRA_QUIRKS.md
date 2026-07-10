# Algebra / OFE elaboration quirks

Recurring patterns when porting non-proof-mode algebra libraries (`Iris/Algebra/*` — OFE,
COFE, CMRA, functors), especially ones built on a pointwise relation such as
`List.Forall₂`, `Option.Forall₂`, or a custom structure. These fixes are **not derivable
from reading the code alone** — they involve typeclass search order, higher-order
unification, and simp's default lemma set. Use this as a checklist when a structural
OFE/CMRA port won't elaborate.

- **Generic helper lemmas must live in the datatype's *root* namespace.** If you build a
  reusable API for a relation (e.g. `List.Forall₂.imp`, `.trans`, `.map`) while inside
  `namespace Iris`, they land in `Iris.List.Forall₂.*` and dot-notation (`h.imp`, `h.map`
  on an `h : List.Forall₂ …`) silently fails to resolve them — you get
  `Invalid field 'imp': … does not contain List.Forall₂.imp`. Open a bare `namespace List`
  (close/reopen `Iris` around the block) so they become `List.Forall₂.*`.
- **`∀ i, l[i]? …` needs the index type annotated.** An unannotated binder leaves the
  `GetElem?` instance search stuck: `typeclass instance problem is stuck: GetElem? (List α)
  ?m …`, because the index type must be fully determined before instance resolution runs.
  Write `∀ (i : Nat), …` (likewise for any `getElem?`/`getElem`/`l[i]!` under a binder).
- **Passing an element-relation hypothesis into a recursive/forwarding call needs the
  relation pinned explicitly.** For `theorem foo (H : ∀ {a b c}, R a b → R b c → R a c) …`,
  a recursive call `foo H …` (or wiring `H` in through `Equivalence.trans`) infers `{R}` by
  *higher-order* unification and picks a garbage solution — the tell-tale symptom is the
  goal's `R` mutating into `fun {a b} => R ?m ?m`. Fix: pass `foo (R := R) H …` at every
  such call. (Same family as the `(F := F)` rule in `IPROP_QUIRKS.md`.)
- **`simp`/`simpa` rewrites a `Forall₂`-cons goal into a conjunction.** Batteries tags
  `@[simp] forall₂_cons : Forall₂ R (a :: l) (b :: k) ↔ R a b ∧ Forall₂ R l k`, so `simpa …`
  on such a goal turns it into `R a b ∧ …` and a subsequent `.cons`/constructor application
  fails with `Unknown constant And.cons`. When the goal head should stay `Forall₂`, use
  `rw [eqn]; exact .cons h t` rather than `simpa [eqn] using …`.
