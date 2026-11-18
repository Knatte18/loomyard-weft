```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens over the diff `72c3e192..HEAD`. For each changed signature, output, key, message, file shape or behaviour, find everything that reads it and say whether the diff leaves it right.

**Per change, the set found and whether it is covered**

1. **`JudgeSnapshot.Changes` now returns `[]GuardChange`, not `[]string`; `checkoutChanges` and `scratchChanges` changed the same way.**
   - `changedSince` (verify.go): updated.
   - `TestJudgeSnapshot` (judgesnapshot_test.go:498-584): updated to compare `.Text`.
   - doc.go (judge-snapshot paragraph): updated.
   - No other caller (grep over `internal/`). Covered.
2. **New `Deps.LockDecisionRecord` and `Deps.ReplacedDecisionRecord` seams.**
   - Production `driftDeps` (loomcli/driftwiring.go:281-285): wired.
   - envkit fixture (envkit.go:749-752): filled.
   - battencli `intentionallyNil` (wire_seams_test.go:202-203): listed.
   - judgesnapshot test fixture: filled.
   - `driftRecordsDeps` builds no guard, so it needs neither seam. Covered.
3. **`TakeJudgeSnapshot` and `Changes` now hold the decision lock.**
   - Every holder of the same lock: the add, store and replace verbs (decision.go:931, 1218, 1327) and `writeCarryOverLocked` (wiring.go:575).
   - None of them calls into drift while holding it, and the guard releases it before `recordGuardHalt`, so there is no re-entry.
   - doc.go and the `Deps` comment are updated. Covered.
4. **Replace-trailer read (`replacedDecisionRecord`).**
   - It uses loomcli's own `recordReplacedTrailerKey` and `parseRecordReplaced` (projectrecord.go:77, 86).
   - The format matches `recordReplacedValue`.
   - Git goes through `fabricengine.TrailerCommits` and `AnchoredFileAt`, which keeps PATTERN-fabric-boundary. Covered.
5. **`SessionMark.GuardHalt` (new JSON key `guard_halt`) and the new `GuardHalt` type.**
   - Writers: `recordGuardHalt` goes through `WriteRecords` and then `state.WriteJSON`, which keeps PATTERN-tolerant-records.
   - `openJudgeSession` (verify.go:591) and `closeJudgeSession` (verify.go:428) replace the whole mark, so a new or closed session drops a stale halt. `resumeJudgeSession` leaves it alone.
   - Readers: `changedSince` and `changeTime`. `driftJudgeSessionOpen` reads only `.Open`. reportengine has its own record type.
   - Covered.
6. **New `Paths.Findings` (`_lym/drift/logs/verify-findings.md`).**
   - The judge guard already exempts `LogsDir` (judgesnapshot.go:399), so writing the findings file is never named as a judge edit.
   - doc.go's Paths paragraph is updated.
   - No test enumerates the `Paths` fields.
   - It is placed under the `durable` root that `NewPaths` derives, keeping PATTERN-lym-files.
   - Covered, except the repeated literal in finding 1.
7. **The no-match halt reason now reads lead, count, way forward, findings, then the file path.**
   - No production code parses a halt reason by splitting at `way forward` (grep over `internal/`; `beamcli/describe.go:78` only checks `Contains`).
   - The halt note is written whole by `internal/haltnote`.
   - Existing `TestDriftVerify` fragments assert substrings that the new text still carries.
   - No stencil, spec or doc quotes the reason (grep over contracts/ and docs/; docs/overview.md:1085 describes Drift-Verify only in general terms). Covered.
8. **The guard-halt reason gains a modification-time suffix and a resume-lead variant.**
   - It keeps the substrings that existing tests assert: "the judge session changed what it may not", the snapshot dir, "keeps the snapshot directory", "lym loom resume".
   - No external reader. Covered.
9. **Commit messages.**
   - 161cab9e's claims match the diff. 80f1b53f and 09ff9d8c are comment and fixture fixes and claim nothing further.
   - There is no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed.
10. **Declared PATTERN changes:** the directive declares none, so there is nothing to land.
11. **Stencil wording:** the diff changes no stencil.

**Findings**

1. **BLOCKING · design · `internal/driftengine/paths.go:69`**
   - Summary: `Findings` repeats the literal `"logs"` already used for `LogsDir` on the line above, with `filepath.Join(durable, "logs", "verify-findings.md")`.
   - Why it matters: this breaks PATTERN-code-conventions "Named constants": a name that can go stale is one named constant and never a repeated literal. Renaming the logs directory would silently leave the findings file outside it, and so outside the judge guard's `LogsDir` exemption (judgesnapshot.go:399).
   - Fix: derive it as `filepath.Join(logsDir, findingsFileName)` from one local or named `logs` value, with the file name as a constant like its siblings `expectedFileName` and `judgementFileName`.
   - Evidence: diff piece 003, paths.go:68-69; pattern/PATTERN-code-conventions.md "Named constants".

There are no other findings in this lens.

**Tool calls**
1. Grepped repo .md/.go/.yaml files for the halt texts and judge-guard phrases, plus `LogsDir` users and contracts/docs mentions of `guard_halt` or `verify-findings`. The context had no search outside the diff's files.
2. Grepped `accepted.go`, loomcli `export_test.go` and `driftwiring_test.go` hits, the `LogsDir` exemption in driftengine, and stencils that mention drift ways forward. The first search named these files, which the manifest had not read.
3. Grepped the parent, darn and recipe stencils and spec for Drift-Verify, judge-guard or drift-amend wording, and listed `docs/`. This checks the stencil and doc member of the halt-text set.
4. Grepped docs/overview.md and logging-inventory.md for judge or Drift-Verify descriptions. Same doc member, for the files the previous call found.
5. Grepped production Go for any parser that splits a reason at `way forward`, and checked for a `Paths` field test. This checks whether reordering the reason breaks a reader the context did not show.
