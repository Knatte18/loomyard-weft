---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/gatecli/scopes_integration_test.go:1110
    summary: The row "a dirty worktree diffs from the parent merge base although the record holds a whole-green ancestor" now passes vacuously, because on a fresh hub its whole-green ancestor is the merge base itself.
    fix: "In that row, commit a change on the pair before the `--all` run, for example `f.commitChange(t, \"other/other.go\", \"package other\\n\\nvar Seen = 1\\n\")` right after `f := newScopesHub(t)`, so the whole-green entry the `--all` run writes is for a commit other than `f.mergeBase`. Keep the later `commitChange` of lib/lib.go, the dirtying `edit` and the `wantDerivation(..., f.mergeBase, \"tier0\")` check as they are."
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: The change description says the plan sits under the never-tracked `.lym` directory, but `planparser.PlanDir` puts it in `_lym/plan`, a weft junction the code repository leaves out through `.git/info/exclude`.
    fix: "In `_lym/landing/summary.md`, `## Design decisions`, bullet \"The plan is written for every scopes row\", replace \"It sits under the never-tracked `.lym` directory, so no row's diff sees it.\" with \"It sits under `_lym/plan`, which the pair's code repository excludes, so no row's diff sees it.\""
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/test_integration_test.go:716
    summary: The `scenarioHub` type comment says every row builds its own hub, but the standalone-worktree rows and the rows outside every hub build none.
    fix: "In the `scenarioHub` doc comment, replace \"Every row builds its own, so rows share no state and run in parallel.\" with \"Every row that runs in a hub builds its own, so rows share no state and run in parallel.\""
---

### [BLOCKING] Dirty-worktree derivation row no longer separates the merge base from the green ancestor

**Location:** `internal/gatecli/scopes_integration_test.go:1110`

**Issue:** At HEAD the row builds a fresh hub. The pair stands on the prime's HEAD, so its HEAD is `f.mergeBase`.
The row's first step, `f.gate(t, f.pair, nil, "--all")`, then records tier0 whole-green at `f.mergeBase` itself.
The row then commits lib/lib.go, dirties docs/note.md and asserts that the derivation line names `f.mergeBase`.
Suppose the gate wrongly used the newest whole-green ancestor on a dirty tree. It would print the same commit, so the assertion cannot fail on the behaviour the row's name rules out.
At the base, the row ran after "a diff run diffs from the newest whole-green ancestor…", which left commits beyond the merge base, up to the lib=8 commit.
Its `--all` run therefore recorded green at a commit other than the merge base, and the check told the two apart.
This breaks the entry's "Keep every assertion" and the rubric's "Tests still check what they say".
The code-correctness lens and the handler found the same defect on their own.

### [LOW] Change description names the wrong directory for the plan

**Location:** `_lym/landing/summary.md`

**Issue:** `planparser.PlanDir` returns `<anchor>/_lym/plan` (`internal/planparser/parse.go:66`, `lymdirs.LymDirName = "_lym"`), not a path under `.lym`.
The conclusion still holds: `_lym` is a weft junction, and the warp's `.git/info/exclude` leaves it out (`internal/hubforge/seed.go:27`), so no row's diff sees the plan.
The stated reason is wrong, though: `_lym` is the tracked records namespace, not a never-tracked one.

### [NIT] scenarioHub comment overstates which rows build a hub

**Location:** `internal/gatecli/test_integration_test.go:716`

**Issue:** Some rows build no `scenarioHub`: "status outside every hub answers in_hub false", "a directory in no git worktree names lym config gate run in one", and the rows that call `standaloneWorktree`.
So the comment's "Every row builds its own" does not match the test it describes.

## Rejected

- code-scope F1, measurements taken without `--gate darn-write`: the recorded entry permits the writer to run a command. It does not require one, and its command cannot run, because `--gate` refuses `--tier` (`internal/gatecli/run.go:414-416`). The change description names the substitute, the reason for it and what it hides. The board entry's measuring item, two runs before and two after with the load at each start, is met.

## Focus verdicts

- Assertion preservation: every row and assertion at the base has a counterpart at HEAD.
  The removed `t.Cleanup`, `defer` and green-record removals served only later rows; the one `t.Error` they held checked a restore write.
  One assertion went vacuous, which is F1.
- Starting state per row: the three rows that read an earlier row's green record now prime it, and each priming run is asserted to run `./...`.
  "a real go test tier in the nested module" creates its own directory.
  "a dirty worktree diffs…" lost the commits it inherited, which is F1.
  The dropped "dirtying note" edits only stopped a carry from an earlier row, and a fresh hub holds no green record.
- Parallel safety: every row builds its own hub, pool, config, green record and plan.
  The shared values `guardInvocation`, `circleTestsArgs`, `tier1File`, `crashEnv` and `stale` are only read, and `crashEnv` is cloned before `append`.
  The go.mod declares Go 1.26, so the loop variables are per iteration.
  The in-process `RunGate` row was already parallel with the package's other tests at the base.
- Refusals table: each row breaks and repairs its own hub, and `setup` and `wantError` take the same `f`.
  The slot-directory row restores the directory before `requireFree` reads `Holders`.
- Tier0 item: not a defect. Every case in these tests runs the gate binary through `startGate` (`internal/gatecli/test_integration_test.go:399`).
  The named candidate, `TestScanGuards_DropsWatchedGuardsNoReadChangeConcerns`, reaches `go list`.
  `withMainPackages` already has its tier0 test (`internal/gatecli/scope_test.go:13`).
- Change description: it states no improvement claim, and it names the load difference that keeps the table from showing one.
  The `two-way` door fits a change to test files alone. F2 covers the one claim the code does not bear out.
- Doc comments: they describe per-row fixtures, apart from F3. No wording about a shared hub or row order is left.
