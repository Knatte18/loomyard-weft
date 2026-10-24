```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's own purpose.

**Decisions walked**
- D1, a `--card` and a computed call under `scoper: dotnet`: the card form reaches the derivation with seeds, and the diff forms reach it once per base. Purpose met.
- D2 diff seed, `Circle.cs` changed in a production project with `CircleTests.cs` naming `Circle`: the closure adds `CircleTests`, the class is selected, and its project is the nearest `.csproj`. Purpose met.
- D2 card seed, a card whose Target is the type glyph `NORCE.Models#Circle`: nothing is seeded and the card runs whole. Purpose missed (F1).
- D3, a changed `GlobalUsings.cs` and a changed `.csproj`: both run whole with a reason. Purpose met.
- D3, a card whose Targets are `docs/x.md` plus one `.cs` file: the `.md` Target yields neither a file nor a type, so every listed tier runs whole. Purpose missed (F2).
- D3, an empty test scope from a non-empty seed, with tier0 (`shared_build`) and tier1 (not opted in) both listed: the record does not say what tier1 does. Purpose unclear (F3).
- D4, tier0 `dotnet test {targets} --no-build {filter}` with `filter: Category!=Slow`, two selected projects: it renders `--filter "(Category!=Slow)&amp;(…|…)"` per project, in one hold. At whole scope it renders the tier filter only. Purpose met.
- D4, a narrowed tier0 handed a SlowTests project with `--no-build` after a build with `-p:RunSlowTests=true`: the test run evaluates `IsTestProject=false`, tests nothing and exits 0. Purpose met.
- D5, `inert_paths: ["*.md"]` with a README-only diff at the computed form and later a whole Merge-In: no tier runs, and the later whole entry carries over the inert-only diff. Purpose met for the diff forms only; the card form is F2.
- D6, card gate tier0 opted in with a one-project narrowed scope, and a build failure: `runListed` → `runGreenAware` → `subtract` → slots held → build → the tier never runs → exit is the build's code. Purpose met.
- D6, an empty test scope: `runListed`'s empty branch takes no slot today, and D6 states the change. Purpose met.
- D7, Linux build and test children with node reuse on: nothing lyx sets defeats reuse. Purpose met.
- D8, timing tier (`slots: all`, `manual`, `carry: false`, no `{targets}`) called with `--tier 9 --wait -- …`: the computed form runs it whole, it is never carried and it writes no entry. Purpose met.
- D9, a documentation-only lens. Purpose met.
- D10, card/batch at `card`, webster at `diff`, Merge-In at `all`: narrowed dotnet passes write no green entry, so webster diffs from the merge base. Merge-In writes a whole entry and Finalize carries it on the same tree. Purpose met only when card and batch gates actually narrow, which F1 and F2 undercut.
- Added-after-Discussion entry, a filter past `greenCommandBudget`: that project runs with the tier filter only. Purpose met.

**Findings**

- **F1 — MEDIUM, design. D2 card-seed bullet (decision-record.md:40) and D3 fallback (decision-record.md:60).** A card Target that is a C# type glyph, or a plain path, seeds nothing, so the card gate runs whole.
  - Evidence: quarry `glyph/glyph.go:23-47`. `NORCE.Models#Circle` parses with `Owner` nil and `Name` "Circle".
  - It is no self glyph (`IsSelf` is false because `Name` is set) and has no owner chain. D2 seeds only "the file a self glyph names, or the type names a member glyph or `plan:` handle spells in its owner chain".
  - So D3 line 60 makes the card run whole.
  - C# has no namespace-level members, so a glyph with an empty owner chain always names a type. It is the most direct way a card can name a class.
  - A plain path Target (`refKindPath`, `planparser/shape.go:250`) is not named as a seed either.
  - Purpose missed: the card and batch gates, which D10 relies on to keep whole runs to Merge-In, run whole for the commonest Target forms.

- **F2 — MEDIUM, design. D3 `--card` fallback (decision-record.md:60) against D5 (decision-record.md:97) and D10 (decision-record.md:173).** Under `--card`, any Target that is not `.cs`, such as a doc file, sends the whole card to the whole solution.
  - D5's inert rule covers only the computed and `--base` forms.
  - D3's card fallback whole-runs any Target that yields neither a `.cs` file nor an owner type, and an inert `.md` Target is one of those.
  - Under D10, card and batch run at scope `card`. A doc-only card therefore runs every listed tier whole.
  - So does a card that edits code plus its docs, which this repo's own docs-in-the-same-commit habit makes common.
  - Purpose missed: Wanted 4's "doc-only diffs skip all test tiers" (board entry) at the most frequent gate. Narrowing is lost on mixed cards.

- **F3 — LOW, consistency. D3's empty-test-scope paragraph (decision-record.md:65-66) and acceptance criterion (decision-record.md:217).** A listed tier that is not opted in has no stated behaviour when another listed tier is opted in.
  - The text says: "runs the build step when a listed tier opts in with `shared_build`, and otherwise runs each listed tier whole".
  - Nothing says whether that other tier runs whole, or runs nothing because it has no invocations (`scopeInvocations` with empty modules, `tiers.go:170`).
  - Walking the mechanism hands the plan writer two readings.

Tool calls: none; the inherited context held every file the walkthrough needed.
