```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced the changed functions on concrete inputs and found no confirmed defects. I listed one unverified low-risk item and two notes below.

**Per function, case run, result**
- `fencedLines`:
  - An opening ` ```js ` followed by a body ` ```js ` stays open, because the rest is not blank. Correct.
  - A 4-space-indented ` ``` ` neither opens nor closes. Correct.
  - A ` ```a`b ` line opens no fence, but `~~~a`b` does. Correct.
  - A closing run shorter than the opening run, or of another character, stays open. Correct.
  - A tab-indented line is not a fence. This is correct for the top-level rule as D4 states it.
  - An unclosed fence runs to the end of the lines. Correct.
  - A CRLF line is trimmed of `\r`. Correct.
- `fenceRun`: an empty line, an all-space line and indent 3 versus 4 all return the right run. Correct.
- `missingSections`: a heading line inside a fence is not counted, so a body `## Open risks` cannot stand in for a removed record section. The fence-opening line is marked fenced and the heading lines are not. Correct.
- `BoardRecord`:
  - An empty body writes an empty fenced block that validates.
  - A body with no trailing newline gets one.
  - A body whose longest backtick run is 3 gets a 4-backtick fence.
  - The `markdown` info string holds no backtick, so the fence opens.
  - Correct.
- `recordWrite.Call`:
  - Record present and log missing: the board is not read, the log is written and the pair is committed. Correct.
  - Record missing and no board entry: Stuck, nothing written, torn files already cleared. Correct.
  - Record missing and log present: the record is written and the log is left untouched. Correct.
  - A crash between the two writes, or before the commit, is completed on resume, because `Commit` runs on every call.
  - `removeTornFiles` matches the `.tmp-` prefix that `fsx.AtomicWriteBytes` uses (`os.CreateTemp(dir, ".tmp-")`, `internal/fsx/fsx.go:60`). Correct.
  - A cancelled context at each step returns the cancellation error. Correct.
- `darnWrite.Call`:
  - Missing record: Stuck before the session, with the goto way forward. Correct.
  - Missing record that HEAD holds: the restore way forward. Correct.
  - Record present and log missing: the goto way forward. Correct.
  - The stop marker is removed before each spawn and read after both a Done and a gate-failed Stuck return, ahead of the guard. Correct.
  - The guard runs only on Done, with no re-prompt, and its reason carries the findings and the restore way forward. Correct.
  - A `Commit` failure, a `CheckRecord` error and a `LatestOutcome` error are returned errors, not Stuck. Correct.
- `checkLoomContract`:
  - A Finalize whose `on_done` successor is a terminal passes, because the walk stops at a non-row name.
  - A Finalize with no `on_done` passes. Correct.
  - A build row before the record row, or off the path, is refused. Correct.
  - The clause order is record, build, order, Finalize, `verify_source`. Correct.
- `decision.go`:
  - `recordWritePending` treats `""`, Preflight and Record-Write as pending. Correct.
  - The missing-record refusal sends an empty row to start and a pending row to resume. A later row goes to restore when HEAD holds a record, otherwise to goto. Correct.
  - The not-committed refusal matches the amended rule: resume at Record-Write, goto then resume later. Correct.
  - On pawl, `RecordEngine` returns `""`, so no record checks run, as before. `HasEngine(pawl, "PlanWrite")` is false, so `--amends-plan` on pawl now refuses, which is correct because pawl has no plan.

**Unverified**
- LOW, design, `internal/loomshed/darnwrite.go:134`. `runAndCommit` returns early when `pointer.Path == ""`, so the stop marker is not read on that path. This only matters if a gate-failed Stuck can come back with an empty path. I did not verify whether it can. If it can, a writer stop would not halt with the description as its reason, as D6 requires "after a gate-failed Stuck". If the session always sets the description path, there is no issue.

**Notes**
- The `Darn-Write` guard commits nothing, so a record the writer edited validly stays uncommitted. A later `decision add` then refuses, and the restore way forward discards that edit. This is as D5 designs it.
- `decisionVerb` reads `found` and `st` at `decision.go:648-660`. I did not see where they are defined, so I assumed they are the status read already in scope.

**Tool calls**
1. Read `internal/loomrecipe/recipekeys.go`, the `fsx.AtomicWriteBytes` temp-file name and the diff of `internal/loomcli/darnwiring.go`, in one Bash call. My context lacked the `HasEngine` and `RecordEngine` behaviour on pawl, the temp prefix, and the Darn-Write seam wiring.
