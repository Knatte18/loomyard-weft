```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the Drift accept diff, one case per changed function or branch.

**List (function or branch: case run: verdict)**

- `testFiles`, name `""` against a baseline holding a top-level dotfile `.gitkeep`: `path.Ext(".gitkeep")` is `".gitkeep"`, the stem is `""`, so it equals `test`. **Wrong.** `lym drift accept ""` passes the check and joins the skip list. See finding 1.
- `testFiles`, name `t1.txt` against the file `t1.txt.bak`: Ext is `.bak`, the stem `t1.txt` matches, so a backup file counts as the test's profile. Correct only for a plain harness directory, and a wider match than "a file named for the test".
- `testFiles`, nested `group/t1.txt` with name `group/t1`: rel is `group/t1.txt`, Ext `.txt`, stem `group/t1`, so it matches. Correct. A bare `t1` does not match. Correct.
- `testFiles`, `dir` absent: returns nil, nil. Correct. A symlink entry is skipped because only regular files count. Correct.
- `requireBaselineTest`, empty or missing baseline: refuses naming the test and the directory, with a way forward. Correct.
- `acceptingCaller`: the wrong-row, operator, agent, blocked and running branches keep the earlier messages and order. The fallthrough keeps the old default message. Correct.
- `admitAcceptance`: the refusals run in the order found, then withdrawn by agent, then caller without ruling, then the baseline name check. Correct. `Accept` runs the check twice, unlocked in `CheckAccept` and again under the scope lock, which is fine.
- `CheckAccept`: read-only. Correct.
- `Accept` and `keepAcceptedOutput`, nothing staged: refuses before any scope write. Correct.
- `Accept` and `keepAcceptedOutput`, `Rename` failing on the second of two files, after `dropAcceptedOutput` already ran: `mutate` returns an error, so the scope is unwritten. The earlier kept output is gone or half replaced, while the scope still lists the old acceptance and the old `AcceptedProfileHead`. Verify then skips the rebuild when HEAD equals that head, so the stale `accepted` profile survives. Otherwise the rebuild builds from a partial or missing kept set. Only I/O failures trigger it, but it is a real inconsistency on a failure path. See finding 3.
- `acceptTest`, `st` read before the capture, which can run for minutes: `Accept` is handed that stale `st`. The agent session can close, or the row can change state, during the capture, and `Accept` then records the acceptance on the old status. The earlier code read `st` immediately before `Accept`. See finding 2.
- `acceptTest`, concurrent accepts: `RunAcceptanceCapture` does `RemoveAll(AcceptingDir)` with no lock and before the slot is taken. A second accept can wipe or overwrite the first's staged output. The first `Accept` then refuses with "no output captured", or keeps output the other capture wrote. No corruption, but the dir is shared. See finding 4.
- `acceptTest`, capture failure or `Accept` refusal: `AcceptingDir` is left in `.lym` holding partial files. The next capture empties it, and `Accept` only reads it after its own capture. Correct, harmless.
- `RunAcceptanceCapture`: timeout, non-zero exit, a file written for another test, a stale file seeded in the dir and a missing capture dir are all covered and correct. The capture skip list is `scope.Skip` only, so earlier accepted tests are captured too. Correct.
- `dropAcceptedOutput`: nothing kept is no error. Correct. It runs inside `Withdraw`'s `UpdateScope`; a failed remove leaves the scope unchanged, but the kept output is partly gone. This is the same class as the `Rename` case above.
- `buildAcceptedProfile`: empty `Accepted` leaves a plain baseline copy. A kept output with a different extension from the baseline file is replaced as a whole. A missing kept output is reported in `missing` and the baseline files are left out. Correct.
- `buildAcceptedProfile`, `CopyFS` into a freshly removed dir: correct. If the baseline dir is absent, `CopyFS` errors, but `Drift-Verify` has already required a counted capture. Correct.
- `keepAcceptedProfile`: nothing accepted or head already recorded gives Done. `missing` non-empty gives a halt and records no head. A failing compare gives a halt and records no head. A pass records head. Correct.
- `keepAcceptedProfile`, scope full: the halt is a decision halt, because `haltReason` prefixes `DecisionHaltPrefix` under full. The caller's re-accept is therefore admitted. Correct. Under scope partial the halt is not a decision halt, but the caller's accept needs no ruling there unless the test is withdrawn. Correct.
- Accepted before this change, or re-accepted after `Withdraw`: with no kept output the row halts naming a new accept. After `Withdraw`, kept is dropped and the re-accept recaptures. Correct.
- `lapse` in `recheck.go`: it clears `Accepted` but leaves the kept files. A later `Accept` replaces them, and the build only reads kept files of listed tests. Correct.
- `Withdraw`: drops the kept output inside the lock and clears the head. Correct apart from the failure-path inconsistency above.
- `CompareAccepted` skipping only `scope.Skip`: accepted tests are compared against their kept output, which is item 2. Correct.
- `siteAcceptanceCapture` in `missingDirRefusal`: names `lym drift accept` as the next step. Correct.
- Removed `CaptureAccepted` seam: no code reference remains. Correct.

**Findings**

1. MEDIUM, `design`, `internal/driftengine/accepted.go:135` (`testFiles`): the stem match admits an empty name against a top-level dotfile, because `path.Ext(".x")` is the whole name. It also lets a stem match across any extension, such as `t1.txt.bak` for `t1.txt`. This can admit a name that is not a baseline test, which is exactly what wanted item 1 forbids. The fix is to take the stem only when Ext is non-empty and the stem is non-empty, and to refuse an empty name. Evidence: the match line at `accepted.go:135`, and `driftcli/cli.go` `cobra.ExactArgs(1)`, which lets `""` through.
2. LOW, `design`, `internal/driftcli/cli.go` (`acceptTest`, the `Accept(w.paths, st, ...)` call): `st` is read before a capture that can take minutes and is reused afterwards. An agent session or row state that changed during the capture is not seen. Re-reading `verbStatus` before `Accept` fixes it.
3. LOW, `design`, `internal/driftengine/accepted.go:158-180` (`keepAcceptedOutput`, `dropAcceptedOutput`) with `internal/driftengine/verbs.go:129-135` and `212-214`: on an I/O failure after the drop, or midway through the renames, the kept output no longer matches the unchanged scope record and its `AcceptedProfileHead`. The accepted profile can then be stale or partial, with no halt on the stale case.
4. LOW, `design`, `internal/driftengine/capture.go:272-277`: `AcceptingDir` is emptied and written with no lock and before the gate slot, so concurrent accepts share it. The loser gets a refusal or keeps output the other capture wrote.

**Tool calls**
1. `grep` of `haltReason` in `internal/driftengine/verify.go`: my context did not hold it, and I needed it to confirm that a scope-full halt is a decision halt.
