```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote matches the code at HEAD.

**Finding 1 — MEDIUM**
- **Changed text:** D4, bullet "Differs from its committed copy" (and the matching D4 acceptance clause): "the way forward is `lym fabric commit` in the `_lym` weft repository, then re-running the verb."
- **Summary:** The fix says re-running `decision add` after `lym fabric commit` makes progress. In the commit-failed case it appends the same entry a second time.
- **Evidence:** In `internal/loomcli/decision.go:700`, the plain path refuses with "the entry is appended but the commit failed … decisionWayFabricCommit". It does not restore the record, so the entry is already in the record. Once `lym fabric commit` lands it, the parent's decision is recorded, and nothing needs to be re-run. `decisionWayFabricCommit` itself names only the commit, not a re-run. In the case where an entry was only appended, the way forward should be the commit alone. A re-run is needed only for an entry the record does not hold.

**Finding 2 — LOW**
- **Changed text:** D4: "takes CommonMark's rule for fenced code blocks. A fence opens on a line of up to three spaces of indent followed by a run of at least three backticks or tildes."
- **Summary:** The fix states the opening rule as all of CommonMark's, but it leaves out one opener clause.
- **Evidence:** CommonMark says a backtick fence's info string may not contain backticks, so a line like ```` ```a`b ```` opens no fence. Under the opener as written, that line opens one. This doesn't affect Record-Write's own fence. It does matter for the claim that the parser takes CommonMark's rule, which D5 relies on for Discussion-written records.

**Claims checked and found to hold:**
- `fsx.AtomicWriteBytes` writes a `.tmp-` sibling, renames it into place, and does not fsync (`internal/fsx/fsx.go:60-77`). It removes its temp file on an error return, so a leftover can only come from a crash.
- Drift-Baseline takes no capture with scope off, `awaited_at` set, or a passed capture. Drift-Await is Done with scope off or `awaited_at` set, whatever HEAD is (`internal/driftengine/doc.go`).
- From a run halted at Darn-Write, `goto --to Record-Write` is admitted, because the target is before the current row (`gotoAdmitted`, `internal/shedengine/goto.go:298`).
- `ValidateLocal` refuses a non-None retired key and names the key without a way forward. `Layers` wraps that refusal with "local config file %s" (`internal/configengine/layers.go`).
- `PrepareRunSettings` reaches `ValidateLocal` through `checkLocalDocument` (`internal/configreg/runsettings.go:135`), after `module.Migrate`. So the claimed position of the separate check, after `moveRunSettings` and before `Migrate`, is consistent.
- The commit-failure refusal names `lym fabric commit` (`decision.go:700`, `decisionWayFabricCommit`), and `lym fabric commit` exists as a verb (`internal/fabriccli`).
- The closing-fence and four-space-indent clauses match CommonMark.

**Tool calls:**
- One grep. It read `gotoAdmitted`, `checkLocalDocument` and the `internal/fabriccli` commit verb, to check the goto admission, where `PrepareRunSettings` validates, and that `lym fabric commit` exists. My context showed only call sites and refusal texts, not these bodies.
