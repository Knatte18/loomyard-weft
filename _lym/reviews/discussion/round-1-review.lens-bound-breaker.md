```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guarantee and claim about existing code that the decision record states.

**Claims attacked:**
- D4, "`shedrecipe.Env` already holds both" the worktree root and the index: **held**. `Env.WorktreeRoot` is at `internal/shedrecipe/recipe.go:88` and `Env.PlanIndex` at `:279`, and `internal/loomcli/wiring.go:550,582` fills both for every row.
- D4, the verb gets the root and index "from its loom wiring": **held**. `validate.go:283-289` already reads `c.env.WorktreeRoot` and `c.env.PlanIndex`.
- D4, "Resolution goes through the code-index seam (`planindex.Index` …)": **broken**. See F1.
- D4, the `glyph-not-found` status policy resolves an ambiguous answer split by build constraint: **held**, per `internal/planglyph/doc.go:36-44`.
- D4, a quarry outage is an error wrapping `ErrQuarryUnavailable`: **held** as a design. The verb has no quarry-specific wording, unlike `validate.go:297-300`, but it is not refused either.
- D4, the glyph shape excludes headings, `C#` and `#123`: **held**. A markdown anchor such as `docs/overview.md#module-table` likely fails quarry's Go member check (`checkGoMember`). I did not confirm that.
- D2, "`lym loom decision add` keeps running `Validate` alone": **held**. `internal/discussionparser/append.go:93` calls only `Validate`.
- D2, "both recipes' gates and `validate-discussion` run them": **held**. `ValidateLoomDiscussion` and `ValidateDarnDiscussion` are the only gate and verb callers (`entries_gate.go:150,155`, `validate.go:70,72`).
- D3, the `_lym` and `.lym` roots come from `internal/lymdirs`: **held**. `LymDirName` and `DotLymDirName` are at `internal/lymdirs/dirs.go:8,13`.
- D3, a token is tested only when its first segment is an entry at the worktree root: **broken**. See F3.
- D3, the path shape rule: **broken** for Go package patterns. See F4.
- D5, the scan sees "a test function that calls `NumField` or `VisibleFields` on a `reflect` value": **broken**. See F2.
- D5, the scan sees `envkit.NilSeams` calls: **broken** inside package `envkit`. See F5.
- D5, the stacking rule against today's guard scan: **held** as a stated change. Today `onlyKeepLinesAfter` (`internal/impactset/guards.go:194-202`) admits only keep lines, and D5 says the guard scan is changed.
- D7, "the tier tags are those the hub's `gate.yaml` declares": **held** for this hub, whose keys `tier0` to `tier3` equal the `-tags` values (hub `gate.yaml:4-38`).
- D8, the coverage forms fit `uses-later-target`, `redundant-package-target` and `card-field-overlap`: **held**.
- D9, "`--require-approved`, which loads no `PlanContext`": **held** (`validate.go:261`).
- D9, webster's dispatch does not run them: **held**. The only callers of `ValidatePlan`, `ValidatePlanLensReview` and `NewPlanGate` are the loomcli verb and `entries_gate.go:166`.
- D9, "skip the cards webster's run record holds done": **held** for the gate. `ValidatePlan` passes only the pending cards (`gates.go:263-269`). `ValidateReworkPlan` passes `plan.Cards` (`gates.go:330`), which is the fresh generation, so this is consistent.
- The constraint that `discussionparser` stays stdlib-only: **held** as a design. The lymdirs filter and resolution sit in loomshed.
- Open risk on in-flight runs: **held**.

**Findings:**

**F1** — MEDIUM, design. Location: D4 ("Resolution goes through the code-index seam (`planindex.Index`, implemented by `internal/planglyph`)").
`planindex.Index` has no method that resolves an arbitrary list of glyphs. Every method takes a `*planparser.Plan` (`internal/planindex/planindex.go:79-102`). D4 reads as if the seam can already do this, but it needs a new `Index` method, which every implementer must then grow, including the test fakes and `envkit`'s `noFindingsIndex` (`internal/testkit/envkit/envkit.go:388`). The Decision should say the seam gains a resolve entry point that carries the `glyph-not-found` status policy.

**F2** — MEDIUM, design. Location: D5 (the scan trigger "calls `NumField` or `VisibleFields` on a `reflect` value").
Most of today's reflective tests call `NumField` on a `reflect.Type`, not a `reflect.Value`:
- `internal/websterengine/config_test.go:49`
- `internal/fabricengine/mergevocab_test.go:205`
- `internal/fabricengine/mergestate_integration_test.go:42`
- `internal/loomengine/config_test.go:1117`

`VisibleFields` is the package function `reflect.VisibleFields(t)`, not a method on a value (`internal/state/keep.go:176`). Read literally, the scan misses these tests, and D5's "config and record structs included" goes unmet. The rule should name both `reflect.Type` and `reflect.Value` receivers and the `reflect.VisibleFields` call.

**F3** — LOW, design. Location: D3 ("its first segment is an entry at the worktree root").
The test reads the worktree root on disk, not the tracked tree. An untracked, machine-local root entry decides whether a token is tested at all. Examples: `.scratch`, which the agent prompt creates on demand, `.dev-bin` from `./deploy-dev`, or an IDE directory. The same record can then pass on one machine and fail with `citation-path-missing` on another. Bounding root entries to tracked ones, or to the `lymdirs` exclusions plus a fixed set, would close this.

**F4** — LOW, design. Location: D3 (the path shape rule).
`internal/...` or `cmd/...`, a Go package pattern a record cites for a Verify command or a scope, has a `/`, no whitespace and none of the excluded runes. Its first segment `internal` exists, so it is tested. It names no file and yields a false `citation-path-missing`. The rule should exclude `...`, or `.` segments generally.

**F5** — LOW, consistency. Location: D5 ("or calls `envkit.NilSeams`").
Inside package `envkit`, the call is spelled unqualified: `NilSeams(FullEnv(t))` at `internal/testkit/envkit/envkit_test.go:17`, which reflects over `shedrecipe.Env`. `NilSeams` lives in `envkit.go:529`, a non-test file, so the "same-package `_test.go` helper" arm does not catch it either. The scan misses the one seam test in envkit's own package, and the acceptance bullet "today's tests the scan sees carry markers" passes over it unmarked.

**Tool calls:**
1. Grepped the callers of `ValidatePlan` and `NewPlanGate` outside loomshed, plus the `lymdirs` constants: my context lacked whether webster or another path runs the plan context checks, and which root names `lymdirs` exports.
2. Located quarry's glyph parser in the module cache: my context lacked the glyph grammar D4 relies on.
3. Grepped `parse.go` for member rules: to test whether a markdown anchor parses as a glyph.
4. Read `parseGo`'s unit and member validation: same reason, to bound F-candidates on anchors and URLs. It was inconclusive, so I raised no finding.
