```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness, run over every function and branch the diff adds or changes.

**What I ran, and the result**
- `editRecordHookCommand("/run/edit-records.jsonl")` gives `lym shuttle hook-edit codex '/run/edit-records.jsonl'; printf '{}\n'`. With lym missing or failing, `printf` still runs and the exit is 0. Correct.
- `hookSettings` with `editRecordsPath` set adds `{matcher="apply_patch",…}` after the activity entry and before the batching and creel entries. Empty path adds nothing. Correct.
- `Prepare` gets `EditLoopEdits=4`. Not interactive, `editRecordsPath` is `<absRunDir>/edit-records.jsonl`, the file `watchEditLoops` reads at `Dir(EventsPath)/EditRecordsFileName`. With `EditLoopEdits=0` or an interactive spec there is no hook. Correct.
- `RecordEdit` with payload `{not json`: parse error returned. Correct.
- `RecordEdit` with `tool_name` "Bash": returns nil and writes nothing. Correct.
- `RecordEdit` with a patch that names no file: `lines` is empty, it returns nil, and no file is created. Correct.
- `RecordEdit` when the records directory is missing: the open error is returned. When the write or close fails: `errors.Join` is wrapped and returned. Correct.
- `RecordEdit` for a patch touching a.go and b.go: stamp, line, stamp, line in one `O_APPEND` write. `ParseSessionSignals` pairs each line with the nearest untaken PostToolUse stamp. Correct.
- `RecordEdit` with `c.now` nil: cannot happen in production. The only production constructor is `NewFromConfig`, which sets `now: time.Now` (`codexengine.go:79`). The hook-edit verb builds the engine through that constructor (`shuttleengines.go:96`). Correct.
- `RecordEdit` on an `apply_patch` call that failed, where the file is unchanged: it records the file with its current hash. Wrong; see finding 1.
- `patchedFiles` with CRLF headers: `\r` is trimmed, and `Index(": ")` lands inside the prefix. Correct.
- `patchedFiles` with "Update old.go" then "Move to: new.go": gives `[new.go]`. Correct.
- `patchedFiles` with a "Move to" before any header: ignored. Correct.
- `patchedFiles` with "Delete gone.go": the path is recorded and the hash is "". Matches the claude unreadable-file case. Correct.
- `patchedFiles` with a.go named twice: recorded once. Correct.
- `patchedFiles` with a relative path and no cwd: skipped. With an absolute path: `filepath.Clean`. Correct.
- `patchedFiles` with a header that has leading whitespace: missed. Wrong; see finding 2.
- `patchedFiles` with `*** Add File: ` and an empty path: `resolvePath(cwd, "")` gives cwd, so the directory is recorded with hash "" because `io.Copy` on a directory fails. A hashless record never counts, so this is harmless.
- `childThreadID` with the session's own rollout (`…-<sessionID>.jsonl`): gives "". Correct.
- `childThreadID` with a child rollout: gives its last 36 characters, the UUID. Correct.
- `childThreadID` with an empty session id or transcript, or a name of 36 characters or fewer: gives "" or the whole name. Correct.
- `fileEditSignal` with `tool_input` missing: indexing the nil map gives "", so it reports false. Correct.
- `fileEditSignal` with `content_hash` missing: `ContentHash` is "", and `fold` skips it. Correct.
- `signalFromPayload` on PostToolUse in the events file: the events file holds no PostToolUse payloads, since the activity hook only touches a stamp. No change. Correct.
- `FileContentHash` move: the body is identical to the removed `fileContentHash`. The claude `RecordEdit` behaviour is unchanged. Correct.

**Findings**

1. MEDIUM, design, `internal/shuttleengine/codexengine/editrecord.go:283-306` (`RecordEdit`).
   `RecordEdit` never checks whether the `apply_patch` call succeeded. It records every file the patch text names, hashing the file as it now stands.
   If Codex fires PostToolUse for a failed or rejected `apply_patch`, the file is still at the content an earlier edit left. Each retry of a failing patch then records that same hash, and `fold` (`editloop.go:106`) counts it as a repeat. A run of failed patch retries would send the parent a false "edited the file back to content it held earlier" notice.
   The claude hook does not have this problem, because Claude Code fires PostToolUse only after a successful Edit or Write.
   - Evidence: `editHookInput` (`editrecord.go:236-245`) reads no tool response or outcome field, and `patchedFiles` works from the patch text alone.
   - Caveat: I could not confirm from the Codex 0.164.0 binary strings whether PostToolUse fires on a failed `apply_patch`.
   - Fix: read the payload's tool response and skip a failed call, or confirm the Codex behaviour and state it in the doc comment.

2. LOW, design, `internal/shuttleengine/codexengine/editrecord.go:326-333` (`patchedFiles`).
   Header lines are matched against the untrimmed line apart from a trailing `\r`. As I recall Codex's own patch parser, it trims a hunk header line before matching. If so, a patch whose `*** Update File:` header carries leading whitespace applies, but this hook records nothing for it.
   - Evidence: the `HasPrefix` checks at `editrecord.go:329` run on `strings.TrimRight(line, "\r")` alone.
   - Caveat: the Codex parser's trimming is from memory, not from a file I read.

**Tool calls**
- Codex binary strings for the PostToolUse fields: to settle whether a failed `apply_patch` fires the hook. The output was inconclusive (the first attempt failed on a regex complexity limit).
- grep for production Codex constructors: to confirm `now` is set on the engine the hook-edit verb builds. My context lacked the registry constructor.
