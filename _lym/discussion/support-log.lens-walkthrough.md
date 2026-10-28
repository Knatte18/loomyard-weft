```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism on one concrete case and checked whether the end state meets the decision's purpose.

**Findings**

1. BLOCKING — D8 against the existing `redundant-package-target` check: a card that targets a package's self glyph cannot pass D8 truthfully.
   - Case: a retag-style Edit card with target `internal/foo#`, in a package that holds `internal/foo/doc.go`.
   - D8 asks for `internal/foo/doc.go` under Edit, Extends or Uses.
   - `checkRedundantPackageTarget` refuses a package self glyph beside a file of the same package in any group except Uses (`internal/planparser/redundancy.go`).
   - So Uses is the only listing that passes. But Uses means "unchanged", and a card that targets the whole package may well change `doc.go`.
   - The way forward D8 names ("list it under Edit") is therefore refused by another check.
   - Fix: D8 should count a package self glyph target as already listing that package's `doc.go`.

2. MEDIUM — D6 meets the same check.
   - Case: a card edits `internal/battencli#` (package self glyph) and adds a field to the marked struct `shedrecipe.Env`.
   - D6 asks for `internal/battencli#TestWire_EverySeamFilled` under Edit or Extends, which `redundant-package-target` refuses.
   - Fix: a package self glyph target, or its `_test` pseudo-unit, should satisfy D6.

3. MEDIUM — D5 against the `//lym:guard` parser's placement rule.
   - The guard parser treats a marker as misplaced unless it sits directly above `func Test…`. Only `//testtiming:keep` lines may come between (`guards.go` `onlyKeepLinesAfter`; the guards_test "stray line" case).
   - D5 puts `//lym:seam` "on the line above it, the way `//lym:guard` sits".
   - Case: a test that is both a guard and a seam test gets `//lym:seam` between the guard marker and the func line, and the guard scan then fails as `GuardMisplaced`.
   - Fix: D5 should either let seam lines sit between a guard marker and its func, or pin an order.

4. MEDIUM — D9 against the Notes and the `planGatePass` signature.
   - The Notes say D6 to D8 fit in `planGatePass`.
   - But `planGatePass` also runs inside `planglyph.Validate` (planglyph.go:96), which is the `--require-approved` path. D9 lists `--require-approved` out.
   - `planGatePass(plan, worktreeRoot, loader)` is not handed the gate config. D7's tier tags from `gate.yaml` live in loomshed's `PlanContext`, and the CLI loads that context only when `--require-approved` is off.
   - Result: either D7 never gets its tiers there (and is silently skipped), or the checks run under `--require-approved` despite D9.
   - Fix: the record should settle which layer hosts D6 to D8 and how the tier set reaches it. It should also state whether `--require-approved` runs them.

5. LOW — D7: the input differs from what the record describes.
   - The plan parser canonicalizes a Create path to a file self glyph (`internal/x/y_test.go#`), so the check never sees "the path as written".
   - The Intent match must use the glyph's unit path or base name. D7 should say that.

6. LOW — D6: Create targets that are `plan:` handles.
   - A field the plan creates is usually spelled as a handle, `plan:internal/x#Deps.NewField`.
   - D6 says "a member of it" but does not say that a handle's unit and member half are matched against the marked glyph.
   - As written, a field-adding Create card can slip past the check.

7. LOW — D4: which directories count as Go packages.
   - The record does not exclude directories the go tool ignores, such as `testdata`.
   - A fixture package under `internal/planglyph/testdata/...` can share a base name with a real package. That would give a false `citation-symbol-ambiguous`.

**Decisions walked**

- D1 — this record's own `### D1 — …` headings, plus an `Added after Discussion` H3: purpose met.
- D2 — this record's acceptance list; D10 and D11 are named, and whole-word matching keeps D1 from matching D10: met.
- D3 — `pattern/PATTERN-test-speed.md`, `go/parser`, `plan:internal/x#Y`, `_lym/plan/`: met.
- D4 — `planparser.GoPackageCounterpart` resolves to `internal/planparser#GoPackageCounterpart`, which exists; `glyph.Self` and `t.Parallel` stay untested because no worktree directory is named `glyph` or `t`: met, apart from finding 7.
- D5 — `TestFullEnv_PathFieldsAbsoluteUnderOneRoot` marked `internal/shedrecipe#Env`: met, apart from the guard-stacking case in finding 3.
- D6 — a card adding a field to `landingshed.Deps` with `TestLandingDeps…` under Uses: met for member targets; not met for package-glyph cards (finding 2).
- D7 — a card creating `internal/x/y_test.go` with "`y_test.go` carries `//go:build tier0`": met if the base name is matched (finding 5).
- D8 — an Edit of `internal/foo#Bar` with `internal/foo/doc.go` under Uses: met; a package-glyph card is not (finding 1).
- D9 — Plan-Write gate and no-flag `validate-plan`: met only if the hosting layer is settled (finding 4).
- D10 and D11 — specs and stencil pointers: met.

**Tool calls**

- Read `internal/impactset/guards.go` and `guards_test.go` to learn the guard marker's placement rule; my context had only the grep hits.
- Read `internal/planparser/redundancy.go`'s head to learn which groups `redundant-package-target` spans; my context had only its name.
- Grepped for `GoPackageCounterpart`, the `planGatePass` callers and directories named `glyph`, `quarry` or `t`, to check D4's mapping and D9's hosting; my context lacked them.
