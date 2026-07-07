# iprop / proof-mode elaboration quirks

Recurring patterns when porting iris invariant-style libraries (`Iris/Instances/Lib/*`
and similar). These fixes are **not derivable from reading the code alone** — they
involve iprop macro recursion, proof-mode tactic syntax, and autobound implicits. Use
this as a checklist when fixing build errors in such ports.

- **iprop forall binders need parens.** `∀ Q : IProp GF, body` *loses* the iprop wrap
  (no `macro_rule` matches), so `={E}=∗` inside `body` parses as a top-level
  `⊢ _ -∗ |={E}=> _` (a Prop). Fix: write `∀ (Q : IProp GF), body`. The matching rule is
  `iprop(∀ ($x:ident : $t), $Ψ)`.
- **`unfold X at Hinv` does not see proof-mode hypotheses.** Replace with `simp only [X]`
  (before `iintro`) or restructure. Symptom: `Unknown identifier Hinv` immediately after
  `iintro #Hinv`.
- **`ispecialize H $$ (term)` fails to parse.** Use `ispecialize H $$ %(term)` — the `%`
  marker is needed for term arguments in proof-mode tactics. Same for `imod H $$ %P` when
  feeding a universally quantified hypothesis.
- **Double-wand `$$ [H1 H2]` is wrong.** For `lemma : A -∗ B -∗ C`, write `lemma $$ H1 H2`
  (no brackets). Brackets are for sep-splitting on a `⊣⊢` direction.
- **`(F := F)` is often needed on every reference to a polymorphic def.** When a `def foo`
  has an implicit `{F}` bound via `variable`, callers may need `foo (F := F) ...` because
  elaboration can't always infer `F` from just the explicit args. Order matters too:
  `(GF := GF) (F := F)` sometimes works where `(F := F) (GF := GF)` doesn't.
- **`⟨One.one⟩` needs a type ascription.** Bare `⟨One.one⟩` passed positionally leaves
  `One ?m` stuck. Always write `(⟨One.one⟩ : Frac F)` (or the relevant type).
- **Pair op is not definitionally equal to the elementwise pair.**
  `(some x, some y) = (some x, none) • (none, some y)` is **not** `rfl`. To split a single
  allocation into two ownership assertions, allocate the op'd form directly rather than
  allocating the combined pair and trying to rewrite.
- **`Timeless (iOwn γ v)` is not an auto-instance.** Construct the needed `OFE.DiscreteE v`
  by hand from the component discreteness *theorems* (they are theorems, not instances).
- **Beware `private` helpers in upstream files.** e.g. a `list_fresh_above`-style lemma may
  be `private`; for a new file needing the fresh-nat trick, inline the bound lemma
  (`hbound : ∀ ys x, x ∈ ys → x ≤ ys.foldr max 0`) rather than importing and calling it.
- **Mask reconciliation pattern for accessor lemmas.** Rocq's
  `iMod ("H" with "[$HP]") as "_". rewrite -union_difference_L //.` ports as:
  ```
  ihave HPor : ... $$ [HP]; · ileft; iassumption
  imod Hcl $$ HPor with _
  rw [subset_union_diff Hsub]
  imodintro; itrivial
  ```
  Doing the rewrite *before* `iapply` breaks: it rewrites `E` everywhere, including in
  `Hcl`'s left mask.
