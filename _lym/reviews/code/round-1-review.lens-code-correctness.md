```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function and branch with a concrete input.

**List**
- `runCaptureChild`, `openWorktree` failure branch (`cli.go:389-393`)
  - Case: cwd is the worktree root, but the anchor is a subdirectory. The envelope reads `{"error":"…does not equal the anchored directory…","lasting_fault":true,"ok":false}`. Correct: `ErrFields` sets `ok` and `error` itself, and the exit code is set.
  - Case: `ReadFile` on the `.lym-anchor` marker fails with a read error other than not-found, for example a transient permission error or I/O fault. The error comes back from `Resolve` through `readRecordedAnchor` (`anchor.go:173-174`) and is marked lasting. That is wrong, but only on a narrow path (see finding 1).
  - Case: `Getwd` fails, or `EvalSymlinks` or `FindRoot` fails (`lymcwd.go:175-181`). A fresh start in the same directory fails the same way, so lasting is right.
  - Case: the marker holds an invalid value, or the stale marker is found. Lasting is right: a fix is needed, and a resume alone does not change it.
- `readChildReport` (`spawn.go`)
  - Case: the log is missing or unreadable. It returns an empty report, so the error names the log only. Correct.
  - Case: the log holds slog text lines followed by one envelope. Non-JSON lines are skipped and the envelope is picked up. Correct.
  - Case: the log holds two envelopes, `A` first and `B` last. `B` wins. Correct: it is the last one.
  - Case: a multi-line error text. `json.Marshal` escapes `\n`, so the envelope is one physical line and the split on `\n` holds. A CRLF log is handled by `TrimSpace`. Correct.
  - Case: a very long envelope line. `strings.Split` has no line limit. Correct.
  - Case: `error` is not a string. The envelope is skipped, and an earlier envelope's report is kept. This is minor staleness, not a defect.
  - Case: `ok:true` or a missing `ok`. It is skipped. Correct.
  - Case: `lasting_fault` is not a bool `true`. `== true` is false. Correct.
- `boundChildError`
  - Case: empty or whitespace-only text. It returns an empty string, so `startFailure` adds no suffix. The `lasting` flag is still carried. Correct.
  - Case: exactly 400 runes. It is returned uncut. Correct.
  - Case: 800 runes. The result is 399 runes plus an ellipsis, 400 in total, cut on a rune boundary. Correct.
  - Case: multibyte text. The cut runs on `[]rune`, so it is rune-safe. Correct.
- `startFailure`
  - Case: no fallback, no report. The text is `…; its log is L`, the same as before. Correct.
  - Case: no fallback, lasting. The text drops the way forward and sets the flag, and the row words its own halt. Correct.
  - Case: fallback, lasting. The text offers no fresh capture and sets the flag. Correct.
  - Case: fallback, not lasting. The old way-forward text is kept, with the child's error inserted. Correct.
  - Case: a stall (`did not record…`) with an empty log. This gives the same text as before. Correct.
  - Case: a stall where the child is still running. Its log is read mid-write. The last line may be partial and is skipped as non-JSON, so the report may be empty. That is harmless.
- `captureAgain` lasting branch (`baseline.go:149`)
  - Case: `startErr` is a `childNoRecordError`, possibly wrapped. `errors.As` with a value target on a value-receiver `Error()` matches. The lasting case is ordered before the plain case, so it is reached. Correct.
  - Case: the reason text. `%v` of `startErr` already ends in "its log is L", and the sentence continues "; lym loom resume cannot change this error…". Well-formed, and it does not offer "resume to capture again".
- `childNoRecordError` is constructed positionally nowhere else. The grep of non-test sources shows only `startFailure` building it, so the struct change breaks nothing.

**Findings**
1. LOW, design. `internal/driftcli/cli.go:389-393`, with evidence at `internal/lymcwd/anchor.go:173-174`. The marking is applied to every `openWorktree` error. One path is not an unchangeable fault: an I/O or permission error reading `.lym-anchor` (`recorded anchor … cannot be read`). A fresh start could succeed there, yet the halt tells the parent that resume is useless. It is narrow, and the other `lymcwd` errors are stable. Consider marking only `ErrNotAGitRepo` and `ErrCwdOutsideAnchor`, matched with `errors.Is`.
2. NIT, design. `internal/driftengine/spawn.go` (`readChildReport` loop). An envelope whose `error` is not a string is skipped silently, so an earlier envelope's text is carried as if it were the last. The child always writes a string, so this is not reachable in practice.

No state-inconsistency or swallowed-error defects found in the rest of the changed code.

**Tool calls**
1. `grep` of `gitWorktreeRoot` and `readRecordedAnchor` in `internal/lymcwd`: my context lacked which errors `Resolve` can return, so I could not tell whether any are transient.
