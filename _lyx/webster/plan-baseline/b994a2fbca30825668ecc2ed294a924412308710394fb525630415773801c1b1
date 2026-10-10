# Card 6 — batten run --window refuses a done slug before opening a window

**Create:**
- `plan:internal/battencli#battenCLI.checkRunStatus` -> `func (c *battenCLI) checkRunStatus() (found bool, err error)`

**Edit:**
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battencli#battenCLI.runInWindow`
- `internal/battencli#battenCLI.doneSlugRefusal`
- `internal/battencli/cli_test.go`
- `internal/battencli/step_test.go`

**Intent:** Decision D4 (#447): `lyx batten run <slug> --window` on a slug whose run is `done` reports ok, opens a window and starts nothing, because `--window` returns before any pre-run hook reads the status and the refusal then fires unseen inside the window.

`checkRunStatus` is `battenPreRun`'s read-only status check, extracted: it ensures the status lock's directory, reads the status file with `state.ReadJSONStrict`, and returns the decode failure, the unrecognized-state error or `doneSlugRefusal()`, with today's messages; a resumable state returns `found` true and a nil error, and an absent file returns `found` false.
It never seeds.
`battenPreRun` calls it and seeds a fresh status only when it reports the file absent, so its behavior is unchanged.
`runInWindow` calls it before opening the window; on an error it prints the error envelope and exits 1 without calling the opener, and on success opens the window as before.
Seeding a fresh status stays inside the window's own run.
`battenPreStep`'s own check, with its refusal-kind mapping, is unchanged.

`doneSlugRefusal` keeps its current way forward and adds the alternative the operator used for #447: give the new task a different slug.
Making a done slug reusable is out of scope; board entry `lyx-cleanup` owns removing finished statuses.
The package doc in `cli.go` states that `--window` refuses a done slug before opening anything.
No refusal names a denied command (`PATTERN-no-denied-recovery`).

Tests:

- `cli_test.go`'s `TestRunInWindow` gives each row's `battenCLI` `shedPaths` under `t.TempDir()`, with an optional status state.
  The existing rows run with no status file.
  New rows: a `done` status exits 1 with the done-slug refusal naming the different-slug alternative, and the opener is never called; a `blocked` status opens the window as before.
- `step_test.go`'s `TestBattenPreStep_RefusalKinds` done row, which asserts the refusal's substrings, also asserts the new different-slug text.

**ImpactSummary:** `runInWindow` reads the slug's status before opening a window and `battenPreRun` reaches the same check through `checkRunStatus`; the done-slug refusal text grows by one alternative, asserted in `step_test.go`.

**Commit:** 6: batten: --window refuses a done slug before opening a window
