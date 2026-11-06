```yaml
lens: code-correctness
outcome: reported
```

**Task restated:** code-correctness lens over the drift-halts diff. Each function or branch below was run by hand with one concrete input.

**Per function or branch, case run, result**

| Function or branch | Case run | Result |
|---|---|---|
| `skipFileArgument` / `withoutSkipFileArguments` | `run --rm -e A={skip_file} -v x img` | Correct: the leading whitespace is consumed and the trailing one kept. |
| same | two adjacent `-e` arguments | Correct: the second pass removes the second one. |
| same | `-e "A={skip_file}"` and `--env=A=…` | Correct. |
| same | `sh -c 'docker run -e A={skip_file}'` | **Wrong, F1.** |
| same | line-continuation command (`img \⏎  -e A={skip_file} \⏎  -v …`) | **Wrong, F2.** |
| `dropSkipFileArguments` | a document with no `command` node, or an empty document | Correct: returns the input unchanged with no lines. |
| `dropSkipFileArguments` | a changed command | Correct: re-encoded through `encodeDocument`, one line per step. |
| `MigrateConfig` | document with no retired key or placeholder | Correct: unchanged. |
| `checkVolatile` | blank file entry, malformed glob such as `[`, blank key | Correct: each is refused with the key named. |
| `Volatile.holdsCase` | pattern without `/` against `a/b/_source.txt` | Correct: matches the base name. |
| `Volatile.holdsCase` | pattern with `/` | Correct: matches the whole path. |
| `Volatile.withoutCells` | zero `Volatile` | Correct: removes nothing. |
| `Volatile.withoutCells` | a cell with an empty key | Correct: never dropped, because blank keys are refused at load. |
| `Volatile.withoutListed` | volatile case | Correct: skipped whole. |
| `Volatile.withoutListed` | volatile key | Correct: the lines are removed from a copy of the slice header. |
| `Check` / `classifyCurrent` | scope has a diff mode but no volatile lists | Correct: keeps empty lists, as documented. |
| `Check` / `classifyCurrent` | scope has no mode | Correct: takes the lists from the config. |
| `ScopeRecord.Volatile` with `omitzero` | `volatile_files: []` loaded as a non-nil empty slice | Not a defect: writes `"volatile":{}`, which is harmless. |
| `lymWrittenRecords` | records uncommitted, or `Mismatch` | Correct: the guarded records are not exempt, so a hand edit halts. |
| `lymWrittenRecords` | both records match the newest `Drift-Records` commit | Correct: they are exempt. |
| `Changes` | lym's own commit of `scope.json` after the snapshot | Correct: exempt. |
| `Changes` | a judge edit of a task file | Correct: still reported. |
| `TakeJudgeSnapshot` | the old record stays while only the copies are cleared | Correct for readers, who never see the snapshot gone. |
| `TakeJudgeSnapshot` | a failure partway through | Leaves the old record beside cleared or partial copies. This is not a defect: the comparison is by hash, and the failure returns an error. |
| `openJudgeSession` | snapshot first, then the mark | Correct: an open mark always has a snapshot. A crash between the two leaves an orphan snapshot and no mark, and the next entry opens a fresh session. |
| `resumeJudgeSession` | `SnapshotLost` true | Correct: clear and commit, then snapshot, so the snapshot holds the commit. |
| `resumeJudgeSession` | crash between the clear and the snapshot | Correct: the next entry finds the snapshot gone and records it lost again. |
| `settleOpenSession` | snapshot found | Correct: goes to `changedSince`. |
| `settleOpenSession` | snapshot gone, no chair | Correct: records lost and returns empty. |
| `settleOpenSession` | snapshot gone, live chair | Correct: records lost and halts with `snapshotGoneReason`. |
| `settleOpenSession` | `Probe` returns an error | Correct: returned before anything is recorded. |
| `pass` after the halt | next entry with `SnapshotLost` true | The guard block is skipped and the session resumes through `resumeJudgeSession`. This is the designed resume. |
| `judgeSessionChanged` | snapshot gone | Returns the reason together with `recordSnapshotLost`'s error. Correct as long as the caller checks `err` first, which I did not verify. |
| `changedSince` | empty `ScratchDir` | Correct: the text omits the scratch part. |
| `changedSince` | non-empty `ScratchDir` | Correct: names the path and keeps the snapshot directory. |
| `CheckRetiredPlaceholders` | drift disabled | Correct: returns nil. |
| `CheckRetiredPlaceholders` | `diff: lines` with `{skip_file}` only in the compare | Correct: the compare is not checked. |
| `CheckRetiredPlaceholders` | `LoadConfig` fails | Returns the load error. This is intended, but see F3. |
| `Preflight.driftFailure` / `prefixedFailure` | nil check, cancelled context, failure | Correct. |
| `wire` | recipe without a `DriftBaseline` row | Correct: `preflightDrift` stays nil. |

**Findings**

**F1 — MEDIUM, design.** `internal/driftengine/diffmode.go` (`skipFileArgument`, the `["']?(\s|$)` tail). The optional closing quote is consumed before the end-of-argument test. `sh -c 'docker run -e A={skip_file}'` matches ` -e A={skip_file}'`, including the quote that closes the enclosing `sh -c` string. The migration then writes a command with an unbalanced quote and reports success, so Preflight no longer finds the leftover. Evidence: `diffmode.go:305-307`. Fix idea: consume a trailing quote only when the opening quote of the assignment was consumed, or leave an argument alone whose quote does not pair.

**F2 — MEDIUM, design.** Same regex, leading `\s+`. In a multi-line command with shell line continuations (`img \⏎  -e A={skip_file} \⏎  -v …`), the match starts at the newline after the first `\` and keeps one space. The result is `img \ \⏎  -v …`. The `\ ` is now an escaped-space argument and a literal space is passed to the container command, which silently changes the command. The tests cover only single-line commands. Evidence: `diffmode.go:305-307`, and the cases listed in the `TestMigrateConfig_SkipFileArgument` additions. Fix idea: when the removed text includes a newline, keep the preceding `\` and the line break, or fall back to the flag (no rewrite) for commands the regex cannot rewrite safely.

**F3 — LOW, design.** `internal/driftengine/config.go` (`CheckRetiredPlaceholders`). It calls `LoadConfig` first, so any unrelated config fault stops Preflight with the "drift config check failed" prefix. That happens even when the fault would never have mattered to the run. The prefix text names only the placeholder in its doc comment, so the reader of the Stuck reason may expect a placeholder. This is arguably intended, and the error carries its own way forward from `LoadConfig`. Raise it only if the decision record wants the Preflight check limited to the placeholder.

**Tool calls:** none. The manifest context already held every definition I used.

I did not verify the `judgeSessionChanged` caller's error ordering, `ReadRecords`' locking, or whether `Amend` and other writers of `scope.Diff` also need to set `Volatile`. The context held none of these and the fork's call budget was spent on none.
