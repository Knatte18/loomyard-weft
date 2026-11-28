```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to the board entry's goal points (code-scope lens).

**Decisions** (the board entry under `## Goal`; the record has no `Added after Discussion` entries)
- G1: add a gofmt check to tier0, so a mis-formatted Go file no longer passes every tier (`Seen` and `Idea`).
- G2 (Orch decision): check every Go file under `cmd`, `internal` and `tools`, not only the changed files, with no diff scope.

**Changes and what they trace to**
- C1: `TestGofmt_ModuleFilesAreFormatted` moves from `cmd/lym/gofmt_test.go` to `internal/testkit/gofmt_test.go` and loses `//lym:only parent-align finalize`, keeping `//lym:guard`. Traces to G1: the guard now runs in tier0 at the task gates and is no longer held for landing. The new third header line saying why the file lives there belongs to C1.
- C2: the whole-module walk stays (`scankit.Walk` with `Filter: scankit.All` and no Roots, plus the `RequireFloor` floor). Traces to G2.
- C3: the `gofmtFinding` message now says `run "lym loom fmt <file>"` instead of "edit the file to match the formatted line", and the first test row asserts `"lym loom fmt a.go"`. No goal point asks for this. It is not forced by C1, since the old message still worked after the move. See F1.
- C4: the `lym loom fmt` help text (`internal/loomcli/fmtverb.go:105`) now names `internal/testkit/gofmt_test.go`. Forced by C1, which made the old path stale.
- C5: `internal/testkit/doc.go:14` gains a line describing `gofmt_test.go`. Forced by C1: that package doc already names its other guard file, `enforcement_test.go`, so the new file is listed beside it.

**Per decision**
- G1: built by C1, with C4 and C5 forced by it.
- G2: built by C2. Every path under `cmd`, `internal` and `tools` is walked except directories whose name contains `testdata`. That exclusion was there before this diff and is the correctness lens's to judge, not a scope gap the diff opened.

**Findings**

F1
- severity: LOW
- class: scope
- location: `internal/testkit/gofmt_test.go:58`, plus the matching assertion in the test row at `:76`
- summary: the finding message's new fix hint, `run "lym loom fmt <file>"`, traces to no goal point and is not forced by the move. The commit message announces it ("A finding now names `lym loom fmt <file>` as the way to fix it") but no `Pattern-Departure:` trailer explains it.
- evidence: `.lym/burler/review-155546164/diff/003-internal-testkit.diff:58,76` against the old line 47 at `.lym/burler/review-155546164/diff/001-cmd-lym.diff:47`; the Goal at `_lym/discussion/decision-record.md:3-18`.
- note for the handler: the hint is small and helps, since agents will now hit this check at every task gate. Keeping it as a scope NIT or rejecting it as part of building a usable tier0 check are both defensible.

No goal point lacks a change, and no other change lacks a trace.

Tool calls: none; my inherited context held the diff pieces, the decision record and the commit log.
