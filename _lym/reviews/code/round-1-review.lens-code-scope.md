```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the goal points of the decision record (#146 and #147) and checked that every goal point has a change that builds it.

**Decisions.** The answer key is the decision record alone. Its `## Decisions` section is empty and it holds no `Added after Discussion` entries.
- **G1** (#146): the halt note records who changed the guarded path, for example by comparing the file's mtime to the judge session's start and end.
- **G2** (#146): either exclude the decision record from the guarded set, or snapshot it under a lock that `lym loom decision add` and `replace` honour.
- **G3** (#146): a resume re-checks the guard against the snapshot and names which path still differs, rather than repeating the first halt's text.
- **G4** (#147): findings that share a case, key and rationale become one entry with a row range and the first and last old/new values.
- **G5** (#147): the halt reason is capped, names the full findings file by repo-relative path, and puts the way forward first.

**Per change, the decision it traces to:**
- **C1, decision lock around the snapshot and every comparison:** traces to G2 (the lock option).
  - `deps.go` adds `LockDecisionRecord`.
  - `judgesnapshot.go` adds `lockDecisionRecord`, used in `TakeJudgeSnapshot` and `Changes`.
  - `driftwiring.go` wires it to `holdDecisionLock`, which add, replace, store and the carry-over writer all hold through their commit (`decision.go:931`, `:1218`, `:1327`; `wiring.go:575`).
- **C2, exemption for a committed replace and an add after one:** traces to G2.
  - `deps.go` adds `ReplacedDecisionRecord`; `appendedDecisionRecord` now accepts two bases; `driftwiring.go` adds `replacedDecisionRecord`.
  - This goes past the lock alone, but it narrows the exclusion option to the parent's two verbs, which is the issue's own reason the halt is abnormal. It is in scope.
- **C3, dating each change by mtime against the session window:** traces to G1.
  - `GuardChange.Path` is set in `checkoutChanges` and `scratchChanges`, and `changeTime` and `changedSince` word the dating.
- **C4, re-check on resume:** traces to G3, and it supplies G1's window end.
  - `records.go` adds `SessionMark.GuardHalt` and the `GuardHalt` type; `verify.go` adds `recordGuardHalt` and the lead "the resume re-checked the judge guard … restored since the last check".
  - `settleOpenSession` and `judgeSessionChanged` now pass the mark, a change forced by this one.
- **C5, grouping ruled cells:** traces to G4.
  - `haltfindings.go` adds `ruledCell`, `groupRuledCells` and `rowOf`, and `checkAndJudge` and `storeRulings` now collect `ruledCell`.
  - It groups by kind as well as case, key and reason. That is a finer grouping than G4 names, not new scope.
- **C6, cap, findings file and way forward first:** traces to G5.
  - `cappedFindings`, `writeFindingsFile` and `Paths.Findings` are new; `noMatchReason` is reordered; `cellsNoMatchReason` is new.
  - The command-mode callers in `passCommand` and `storeCaseRulings` were changed to fit the new signature, which also gives them the reorder. That is forced and also in G5.
- **C7, `doc.go` paragraphs on the lock, the replace exemption, dating, re-check, grouping and the findings file:** forced by C1–C6.
- **C8, tests:** forced by C1–C6.
  - In `judgesnapshot_test.go`: four replace rows plus the lock assertion.
  - In `verify_test.go`: the many-cells halt case and the resume case.
- **C9, nil-seam entries in `battencli/wire_seams_test.go` and seam fakes in `testkit/envkit/envkit.go`:** forced by C1 and C2, whose new seams `wire()` leaves nil for batten.
- **C10, commit `80f1b53f` re-breaking comment lines:** forced by the gate's comment lint. The range's diff stat shows the twelve files above and nothing else.

**Per decision, the changes that build it:**
- G1: C3, with C4 supplying the window end.
- G2: C1 and C2.
- G3: C4.
- G4: C5.
- G5: C6.

**Findings:** none. Every change traces to a goal point or is forced by a traced change, and every goal point has a change that builds it.

One note outside this lens: `haltReasons` (`loosened.go:170`) still appends the uncapped loosened-check reason after the capped no-match reason. G5's cap therefore covers the ruling findings but not every possible joined reason. That is for the correctness lens or the handler to judge.

Tool calls: none; everything I needed was already in the inherited context.
