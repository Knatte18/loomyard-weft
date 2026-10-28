# Card 5 — code-review-lenses

**Edit:**
- `internal/lenses/template.yaml`
- `internal/lenses/sets/csharp.yaml`
- `internal/lenses#TestConfigTemplate_DecodesAndIsSelfConsistent`
- `internal/lenses#TestCheckLoad`
- `docs/dotnet-hub.md`

**Intent:**
D3: delete `code-pattern-fit` from `internal/lenses/template.yaml`; `review-code` becomes `code-correctness`, `code-affected-set`, `code-tests`, and `internal/lenses/sets/csharp.yaml`'s `review-code` becomes those three plus `numerics`.
`code-affected-set`'s set gains the repository's rule documents that describe a changed behaviour: the `PATTERN.md` entries and `pattern/` background files where the repository has a `PATTERN.md`, otherwise the rules its `CLAUDE.md` files state.
It applies, where this round's rubric carries them, "Stencil wording", "The repository's own rules", "Declared PATTERN changes landed" and "Commit messages and departures", and keeps its list, per change, the set found, covered or not.
"A broken invariant" and "A pattern the parent changed", which no rubric carries, are dropped with `code-pattern-fit`.

D4: `code-tests`' prose becomes: for each behaviour the diff adds or changes, find the test row that would fail if it were broken; a behaviour with no such row is a finding, a stencil's static prose exempt; "Your list: per behaviour, the test that pins it, or none."
It no longer opens the existing tests of each new row and names no "Test economy"; `contracts/stencils/loom/loom-rubric-code-review.md` is unchanged, so the review lead still judges test economy.

Tests in `internal/lenses`:
- `TestConfigTemplate_DecodesAndIsSelfConsistent`: `wantLenses` becomes 37; `review-code` 3 entries and its order `code-correctness`, `code-affected-set`, `code-tests`.
  `wantSentences` gains `code-affected-set`: "PATTERN.md", "Declared PATTERN changes landed" and "Commit messages and departures", and `code-tests`: "Your list: per behaviour, the test that pins it, or none."
  A check beside `wantSentences` fails when `code-tests`' text contains "Test economy".
  The test also asserts that `cfg.Lenses` does not define `code-pattern-fit`.
  The `csharp` layer needs no new assertion: its subtest already requires a layer fan to start with the base fan and end with `numerics`, and `CheckLoad` under `lens_set: csharp` fails on a leftover `code-pattern-fit`, which no layer defines.
- `TestCheckLoad`'s row "a remove emptying a fan passes the check and the fan is refused at resolution" drops `code-pattern-fit` from its `remove` list, which then names the three lenses of `review-code`.

Docs (D7): `docs/dotnet-hub.md`'s `review-code` example drops `code-pattern-fit`, leaving the template's three lenses and `models-rules`.

A hub whose `lenses.yaml` names `code-pattern-fit` in a fan a fan key names fails every loom config load after the deploy; the record's open risk names the Models hub's operator dropping it first.
The refusal it meets says to define the fan or set the entry to `""`; the recovery is to drop `code-pattern-fit` from the fan with `lym config lenses --set` or the editor, and the shared refusal text stays as it is.

**ImpactSummary:** the template and the csharp layer, two tests in `internal/lenses` and one doc example; no Go production code or stencil names `code-pattern-fit`.
