```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code (F1 to F6) by running concrete inputs through it and asking whether each added test fails with its defect in place.

Result: no findings.

Traces I ran, from the context alone:
- **`skipFileArgument` and `argumentRemainder`, `run \⏎  -e A={skip_file} \⏎  -v …`:** the match starts at the newline, with lead `"\n  "` and end `" "`. The lead holds a newline and the end is not `"\n"`, so the remainder is the lead. The result is `run \⏎  \⏎  -v …`, which equals the test's `want`. With the old `$1` replacement it was `run \ \⏎ …`, so the row fails with the defect in place.
- **`sh -c 'docker run -e A={skip_file}'`:** the unquoted alternative needs `\s|$` after `{skip_file}`, but `'` follows. The quoted alternatives need an opening quote right after `-e `. Nothing matches and the command stays whole. The old trailing `["']?` ate the quote, so the single-quoted row and the double-quoted twin both fail on the old code. The `tt.want == tt.command` branch asserts no lines.
- **Re-matching a match with `FindStringSubmatch`:**
  - Leading whitespace: `^` matches empty at position 0, then `-e` fails, so the engine backtracks to `\s+`. The groups come out right.
  - Match at string start: lead is `""`.
  - Match at string end: end is `""`.
  - The result is never `nil`, because the string matches itself.
- **Adjacent or continued arguments:**
  - `-e A={s} -e B={s}` takes two passes of the outer loop, as before.
  - Two continuation-form arguments in a row also work, since the lead starts at the `\n` and not at the `\`.
- **Package var init:** `skipFileArgument` depends on `skipFileAssignment`, which Go orders by dependency; `skipFilePlaceholder` is a const.
- **F4 (snapshot failure):** `Checkouts` is called only in `judgesnapshot.go` at lines 140 and 197, so the injected error first hits `TakeJudgeSnapshot` inside `openJudgeSession`. `Call` returns that error. With the mark opened before the snapshot, `JudgeSession.Open` would be true and the row would fail. `f.seats.Calls == 0` also holds.
- **F6 (expect pass):** `f.cfg.Volatile` is set before `seed`. `seed` does not set `Volatile`, and `deps.Config` reads `f.cfg` at call time. Dropping `Volatile` from `expect.go:125` leaves `scope.Volatile` empty, so the `reflect.DeepEqual` check fails.
- **F5 (run settings):** the two refusal strings use the same format as the existing `diff` and `key_patterns` rows, and the gate passed.
- **F1 (constant):** the only `"DriftBaseline"` literal left in `loomcli` is gone. The `const entry` in `shedrecipe/entries_drift.go` is a separate package and out of scope.

Further tool calls: none. My context held every file I needed.
