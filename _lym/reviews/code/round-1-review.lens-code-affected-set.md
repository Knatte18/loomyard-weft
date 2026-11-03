```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, covering what the `WaitNote` signature change, the change key and the watch wording force outside the diff's own lines.

**Per change, the set found:**
- **`Hooks.Waiting` now returns `WaitNote`.**
  - The only production assigner is `internal/loomcli/arm.go:439` (`reviewWaitingFor`), which the diff changes.
  - The only production readers are `waitingNote` in `internal/shedverbs/status.go:101`, the `statusCmd` use at `status.go:215-225`, `statusWatchLine`, and `checklistPoll` at `internal/loomcli/checklist.go:168`. All use `.Text` or `.Steady` consistently.
  - The tests that assign or call the hook (`reviewstatus_test.go`, `status_test.go`) are updated. A repo-wide grep for `Waiting` found no other hook consumer. Covered.
- **`PrintStatusLinesOnChange` now takes `poll func() (line, key string)`.**
  - The only callers are `runStatusWatch` (`status.go:113`) and `status_test.go:368`. Both are updated. Covered.
- **`statusWatchLine` now returns `(line, key)`.**
  - The only callers are `runStatusWatch` and `status_test.go:772-788`. Covered.
- **`UnavailableLine` keeps its signature.** `checklist.go:204` uses it only as display text, so it is unaffected. Covered.
- **Wording about what the watch prints.**
  - Updated: `contracts/specs/loom-status-spec.md:131`, `docs/overview.md:557`, the loom `status` verb help at `internal/loomcli/cli.go:387`, the `--watch` flag help, `status.go:3`, and the `spec.go` comments.
  - `internal/battencli/cli.go:249-251` still says "printing a line only when the composed activity changes rather than once per poll". That is still true: batten has no `Waiting` hook, so the sentence is accurate. Left as is, correctly.
  - `internal/shedverbs/human.go:29` and `RenderStatusHuman` take the text only, and the diff passes `.Text`. Unaffected.
- **Envelope `waiting` key and status file schema.** `core["waiting"] = waiting.Text` is unchanged in value. No schema change. Covered.
- **PATTERN entries.**
  - `PATTERN-cli-cobra` and `PATTERN-daemon-wakeups` name the `--watch` tails, but only for the wakeup and handoff rules, which this change does not touch.
  - `PATTERN-spawn-observability` and the other entries are not touched.
  - The directive declared no PATTERN changes.
- **Commit messages and departures.** No commit carries a `Pattern-Departure:` trailer, and none is needed.

No findings.

**Tool calls (2):**
1. A grep for `composed activity|once per poll|--watch|waiting note|waiting key` over md, go and yaml. My context lacked the docs, stencils, other verb help and `pattern/` entries that describe the watch.
2. A grep for `UnavailableLine|statusWatchLine|runStatusWatch|PrintStatusLinesOnChange`. My context lacked the full caller set of the changed signatures.
