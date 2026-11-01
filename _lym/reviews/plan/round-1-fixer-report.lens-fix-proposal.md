```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the reviewer's proposals the fixer carried out as proposed (F1, F5–F12).

**MEDIUM: card 14, chair stencil Step 4, the new F10 bullet "In `## Shared Decisions`, rewrite each carried decision's `Applies to` line to the joined plan's card numbers, which `lym loom join-parts` printed for each part's cards."**
- **Summary:** The numbers `join-parts` printed are stale once Step 4's first bullet has inserted shared-spot cards with `lym loom insert-card`, so the chair writes wrong card numbers.
- **Concrete case:** The join moves part B's card 1 to joined card 4. The chair then inserts a spot card at position 4, as Step 4's first bullet tells it to, which shifts B's card to 5. The chair then rewrites B's decision as "Applies to: cards 4" from the printed list, which points at the spot card.
- **Evidence:** `insert-card` does not fix bare `Applies to` numbers.
  - `internal/planparser/renumber.go:122`: InsertCard "adds the card's line to the overview's Card Index"; it rewrites only `&lt;N&gt;-&lt;slug&gt;` card ids.
  - `internal/planparser/renumber_test.go:298` asserts "overview changed outside the Card Index" against an overview whose Shared Decisions hold `- **Applies to:** cards %d, %d` (line 51).
- **Correct form:** The bullet should take the numbers from the overview's Card Index as it stands after every spot card is inserted, matched by slug, rather than from the printed join list.

The other proposals (F1, F5, F6, F7, F8, F9, F11, F12) gave no concrete breaking case, so I have no findings on them.

Tool calls:
- grep of `internal/testkit/envkit` for `Setting` and of `LoadTapestryConfig`: to check that the F1 `env.Setting("tapestry", "parts_max")` lookup is answered in `FullEnv` (it is, via `TemplateSetting`). My context did not have envkit's Setting wiring.
- grep of `internal/planparser/renumber*.go` for overview and `Applies to` handling: to test whether the F10 numbers survive `insert-card`. My context did not have the renumber code.
