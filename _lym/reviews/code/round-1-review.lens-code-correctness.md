```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens review of the diff.

**Per-function list**

- `store.go validateWrite` note-dependency refusal: ran `depends_on ["dep","idea"]`, slug `x`. It clones the list and drops `idea`, leaving `["dep"]`. The set-deps argument is valid JSON. Correct.
- Same refusal with only `["idea"]`: `DeleteFunc` leaves a non-nil empty slice, which marshals to `[]`, not `null`. Correct. With `["idea","idea"]` both copies are removed. Correct.
- The `json.Marshal` error is discarded. A `[]string` cannot fail to marshal. Correct.
- `check.go dependencies()`, dependency that is a part: the first case matches and no finding is raised. Correct. A slug that is both a part and a board note counts as the part, which the summary says is intended.
- Dependency on a board note: `depends-note` is raised. Both `BoardEntry` producers fill `Kind` (`internal/loomshed/raddlefile.go:538`, `internal/loomcli/raddlecontext.go:88`), so the check fires in practice. Correct.
- Dependency on a note that is also in `board_checked`, with `Filing` true: `depends-note` is raised, not the gone-entry pass, and a test row pins it. Correct.
- Dependency on a `board_checked` slug that is gone from the board: `entry < 0 && !checked` is false, so it passes as before. Correct.
- Dependency on a slug that is nowhere: `depends-unknown` is raised as before. Correct.
- `loop.go appendLoopChildOutput`, empty stdout: returns early. Correct.
- Same function, `"first\nsecond"` with no trailing newline: the output is two prefixed lines, each ending in `\n`. If the existing log ends without a newline, a leading `\n` is added first. Correct, and a test pins it.
- Same function, stdout `"\n"` (one bare newline): `TrimSuffix` leaves `""`, so one line `…output: ` with empty text is written. Harmless, because `loopLogLinesOf` drops it by prefix. Correct, though it conflicts with the doc line "Empty output keeps nothing".
- Same function, stdout with blank lines in the middle: each becomes a prefixed line with empty text. Correct.
- `loopLogLinesOf`, log with a start mark, own marks, child lines and own lines: marks go to `marks`, child lines are skipped, own lines go to `output`. Correct, and a test row pins it.
- `loopLogLinesOf`, no start mark: `started` is false and no lines are returned. Correct. An unreadable or missing log gives the same result, so the "no start mark" reason is slightly misleading there. This is LOW.
- `loopLogLinesOf`, a child line that starts with `lym shed: loop <id> starting`: it now sits behind the `lym shed: step` prefix, so it can no longer pass as a mark. This is an improvement.
- `loopExitEvidence`, three branches: marks present, no start mark, start mark but no exit mark. Each message matches its test row. Correct.
- `loopProcessExitRecorded`: it ignores `started` and its behavior is unchanged. Correct.

**Findings**

1. LOW, design, `internal/shedverbs/loop.go:~757` (`appendLoopChildOutput`). The doc says "Empty output keeps nothing", but stdout that is only `"\n"` writes one bare `lym shed: step <id> output: ` line. Only `len(stdout)==0` is checked. The line is invisible to the excerpt, so the only effect is log noise.
   - Evidence: the `len(stdout) == 0` guard and the `TrimSuffix`/`Split` path.
   - Fix: also return early when `TrimSuffix(string(stdout), "\n") == ""`, or word the doc as "no output".

2. LOW, design, `internal/shedverbs/loopwait.go` (`loopExitEvidence`, the `!started` case). `loopLogLinesOf` discards the `os.ReadFile` error. A missing or unreadable log therefore reports "the loop log holds no start mark of loop X", which is not the real reason, and the stated cause is wrong.
   - Evidence: `data, _ := os.ReadFile(logPath)`.
   - Fix: have `loopLogLinesOf` also report whether the read failed, and name that reason in the clause.

3. LOW, design, `internal/shedverbs/loop.go` (`loopChildOutputPrefix`) and `loopLogLinesOf`. A line of the loop's own output that starts with `lym shed: step ` is dropped from the excerpt. The summary acknowledges that an old log with unprefixed child envelopes still shows them, and nothing else is wrong. Accepted risk, not a defect; reported for completeness.

No finding on the raddle or board paths. The design (a shared check, with the part winning over the note) is consistent.

**Tool calls**
1. Read `loopLogLinesOf` in `loopwait.go`, the `store.go` imports and the surrounding refusal code. The diff showed only partial context.
2. Grepped the `BoardEntry{` producers and `loopLogMarkPrefix`, to confirm `Kind` is populated and to read the prefix value.
3. Read `raddlecontext.go:80-100`, to confirm the `Kind` and `Open` mapping.
