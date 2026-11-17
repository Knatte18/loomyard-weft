```yaml
lens: code-correctness
outcome: reported
```

Lens code-correctness: I ran each function and branch the diff adds or changes on a concrete case, and found one NIT and no defect in the shapes the spike rollout shows.

**Per function and branch: the case I ran, and whether it is correct**

- `bashlookup.LineRangeFile`:
  - `sed -n 1,2p a.go 2&gt;/dev/null` gives `a.go, true`, because the `/dev/null` redirect is dropped. Correct.
  - `sed -n 1,2p a.go; cat b` gives false: a second non-empty segment. Correct.
  - `""` and an unterminated quote give false. Correct.
- `sedRangeFile`:
  - `sed -n -e 1,2p a.go` gives `a.go`. Correct.
  - `sed -n --expression=1,2p a.go` gives false, because the script hides in an option and only one operand is left. This is a false negative only, so no breach is invented. Correct.
  - `sed -n -f s a.go` gives false: the regex fails on `s`. Correct.
- `countedLinesFile`:
  - `head -n` with no value gives false (the `i+1&gt;=len` check). Correct.
  - `head -n5`, `--lines=+5` and `tail -n +20` give the file. Correct.
  - `head -c50`, `head -`, `head --` and `tail -f` give false. Correct.
  - Two files give false. Correct.
- `IsReadOnlyLookup` rename: the behaviour is unchanged, and `claudeengine` now calls `bashlookup.IsReadOnlyLookup` at both of its sites (`claudeengine/batchingnudge.go:249,360`). Correct.
- `ReadFileTail` and `CountLinesUpTo` moved without change. `CountLinesUpTo` with limit 0 returns 0, but validation rules out `slice_lines` below 1. A directory path errors on Read, which means no breach. Correct.
- `rolloutCalls`, against the real 2026-11-16 spike rollout:
  - The `item_completed` `CommandExecution` line carries `command` as `[]string`, `cwd` as a `file://` URL and `parsed_cmd[].path` as `notes.txt`. The `FileChange` carries the id `exec-…`.
  - The order is `custom_tool_call`, then the item, then `custom_tool_call_output`, for every cell.
  - The struct decodes all of these. `UserMessage` and `AgentMessage` items reach `default: continue`, which continues the `for` loop. Correct.
  - An item with no open cell is skipped, and `slices.Max` only runs when `len(open)&gt;0`. Correct.
  - The output of a `function_call` deletes a key that is absent, which does nothing. Correct.
  - A cell whose output is never written, such as one cut off by an aborted turn, stays open. The items of later cells go to the open cell with the highest index, which is the newest. Correct.
- `shellCommandOf`: `["/bin/bash","-lc","cat notes.txt"]` gives `cat notes.txt`. An argv with no shell is joined with spaces. Correct.
- `localPath`: `file:///home/x/work` gives `/home/x/work`, and a plain path passes through. Correct on POSIX. On Windows it is wrong; see F1.
- `resolvePath`:
  - A relative path with cwd `""` gives `""`, which never equals a resolved path. Correct.
  - `../long.txt` against `dir/sub` gives `dir/long.txt` after the join cleans it. Correct.
- `BatchingNudge`:
  - Malformed JSON is an error. Correct.
  - A missing `transcript_path` or `tool_use_id` gives nil. Correct.
  - `collaborationspawn_agent` returns before reading the state. A non-lookup Bash call also returns before the state. Correct.
  - A state file that is absent, or a thread at its cap, gives nil without waiting. Correct.
  - `UpdateJSON` returns the mutate error unwrapped (`internal/state/state.go:173-175`), so `errors.Is(err, errNoNudge)` holds. Correct.
- `awaitRollout`:
  - Attempt 0 always reads, because `readSize` starts at -1, and the context gate applies there. Correct.
  - With attempts 3 and the item never arriving, the loop sleeps 3 times and returns the last tail with `true`, as D12 asks. Correct.
  - A stat error or a read error gives false. Correct.
  - A tail that is all one partial line gives nil, so the context is unknown and the call is silent. Correct.
- `latestExecCell`, `withCall` and `holdsItem`:
  - When an item is not recorded yet, the payload call is added to the latest cell as unrecorded. Correct.
  - When the item is already recorded, the recorded copy, which carries cwd and reads, is kept. Correct.
- `kindOf`:
  - A cell with one patch counts as edits; one with patch and lookup counts as other; an empty cell counts as other. Correct.
  - A non-exec call counts as other, so a `function_call` ends both chains. Correct.
  - A single `cat a &amp;&amp; wc -l a` command counts as one lookup, the same as Claude's single Bash lookup. Correct.
- `judgeBatching`: `IndexFunc` cannot return -1, because `withCall` guarantees the current call is in the latest cell. Correct.
- `sliceBreach`:
  - A read that is not recorded gives `""`. Correct.
  - A short file gives the short reason. Correct.
  - An earlier read in a cell at or after the mtime gives the read-again reason; an earlier read before the mtime gives `""`. Correct.
  - An earlier read whose cwd is `""` never matches. Correct.
  - A read in the same cell is not counted, as D12 says ("earlier cell"). Correct.
- `batchingNudgeHookCommand`:
  - When lym is missing, `out` is empty, so the command prints `{}` and exits 0. Correct.
  - The hook's stdin passes into the `$(…)` that reads it. Correct.
- `batchingHookOutput`: the JSON shape matches the one the test reads. Correct.
- `questionDenyCommand`:
  - A `tool_name` of `functionsrequest_user_input_async` is denied; `collaborationspawn_agent` is not. Correct.
  - A command that only mentions the name is escaped as `\"tool_name\":`, so the pattern does not match it. Correct.
  - The steer has no quote, so it is safe inside the single-quoted echo. Correct.
- `denyNotice(true, nil)` gives the question notice alone. Correct.
- `hookSettings`:
  - With no deny and no guards there is no PreToolUse setting. Correct.
  - The deny comes before the guards. Correct.
  - PostToolUse runs the activity hook, then batching, then creel. Correct.
- `Prepare`:
  - `questionDeny` is `!spec.Interactive`. Correct.
  - The state file is written only while the nudge is enabled, and a write error stops the launch. Correct.
  - `OpenLaunch` does not call `hookSettings`, so it is unaffected. Correct.
- `validateBatchingNudge` and `LoadConfig`: a block that is off skips the checks. `context_tokens` of -1 is refused, naming the key; `lookup_run` of 0 is refused. Correct.
- `parseRolloutEvents` and `latestContext`: the same loop, moved, and `ContextTokens` gives the same result. Correct.
- The seam scan `sharedPackages`: `…/shuttleengine/bashlookup/x` cuts to `bashlookup` and is exempt. `…/claudeengine` is still checked. Correct.

**Findings**

- **F1 — NIT, design.**
  - Location: `internal/shuttleengine/codexengine/batchingnudge.go:305-310` (`localPath`).
  - Summary: on Windows, a `file:///C:/x` cwd becomes `\C:\x`, which is not absolute. The read paths then resolve to paths that do not exist, so the read-slice breach never fires there and nothing says so.
  - Evidence: `url.Parse` puts the drive in `Path` as `/C:/…`, and `filepath.FromSlash` keeps the leading separator (`batchingnudge.go:307`). `codexengine/codexengine.go` branches on `runtime.GOOS`, so the engine is built with Windows in mind.

There is one premise I could not check. Codex might write a nested call's `item_completed` only after the PostToolUse hook returns. If so, every line-range read waits the full 2 s bound (`batchingRolloutTick`×`batchingRolloutAttempts`) and never gets a read-slice nudge. The spike rollout only shows that the item precedes the cell's output. D12 sets this design, so I have not filed it as a finding.

Tool calls:
1. Pulled the `item_completed` and `custom_tool_call` lines of the 2026-11-16 spike rollout, to check `rolloutCalls`' decode struct against real line shapes. My context had only D12's prose.
2. Listed the call and item order and read a `FileChange` line of the same rollout, to check how cells nest and the shape of the patch item, which my context lacked.
3. Grepped `codexengine` for GOOS handling and read `state.UpdateJSON`'s error returns, to check Windows reach and whether `errors.Is(errNoNudge)` holds. Neither was in the manifest.
4. Looked for the spike's hook log, or a research note on whether the hook or the item comes first, to check the item-wait premise. The spike directory is gone and no report states the order.
