Task: check the fix set for stale twins and sibling text that disagrees with the changes.

**Findings**

1. **LOW** — `internal/driftcli/amend.go:42`, the `lym drift amend` help text.
   - Summary: The help text still lists the admitted rows as "Darn-Write, Webster, Drift-Verify or Drift-Recheck", but the fix added Drift-Expect to `amendRows`. The same help text is also what `lym drift amend` shows a parent.
   - Evidence:
     - `internal/driftengine/amend.go:32` now holds `expectRow` in `amendRows`.
     - The `amend.go` and `doc.go` doc comments were updated for it.
     - The CLI `Long` text is the one twin that was left as it was.
     - The new halt `recordsMismatchHalt` sends the parent to run `lym drift amend <slug> --restore` at Drift-Expect, and the help text tells that parent amend is not admitted there.

2. **LOW** — `internal/driftengine/records.go`, the new `OpenedAt` comment, against `internal/driftengine/verify.go:362`.
   - Summary: The `SessionMark` type doc now says "a zero mark [records] one never opened", but the judge mark's close still writes the zero `SessionMark{}`.
   - Evidence:
     - `verify.go:362` has `scope.JudgeSession = SessionMark{}` in `closeJudgeSession`.
     - A reader of the type doc would take a closed `JudgeSession` as a judge session that was never opened.
     - The new rule holds only for `ExpectSession`.
     - Either narrow the comment to the expect mark, or keep `OpenedAt` on the judge close as well.

Checked and consistent (no finding):
- `doc.go`: the gate passes (lines 136–139), the amend rows (155), the Drift-Expect mismatch halt and the ended mark (199–200), and the key-group sentence (64).
- `ListCaptures` doc: matches the empty-key rule in `listLines`.
- `deferHaltRows` and `verbs.go`: they concern Defer, not Amend, so they are correctly unchanged.
- `TestDefer`: no other reference to `TestDeferWaive` remains.
- `check.go`: the `NewDriftGate` doc agrees with the `clearable` line.
- `expect.go`: the `Call` and `runAgent` docs agree with the code.

Tool calls made:
1. Bash grep for the amend row lists and the session-mark wording across `*.go`, `*.md` and `*.yaml`. The context lacked any search outside `internal/driftengine`.
2. Read `internal/driftcli/amend.go` lines 30–59. The grep showed the help text and the context lacked the file.
