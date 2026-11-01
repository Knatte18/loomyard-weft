```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens review of the plan's refusal, stop and recovery ends.

**Findings**

1. **MEDIUM** — Card 11, start failure, D1. When the runner cannot start for a reason that persists, the way forward cannot succeed, because no route out of the loop is named.
   - The start failure exits 1 with "way forward: run the same command again".
   - When the cause persists, the same command starts another runner that fails the same way, every time. Examples: `os.Executable()` failing, a binary that will not exec, a runner that crashes at once, or `launchRunner` refusing under `testing.Testing()`.
   - The front's text never names `--wait`, which D2 runs in-process and would progress.
   - Fix: the start-failure envelope names `--wait` as the fallback after a repeat failure.
   - Evidence: card 11's Front paragraph (start-failure sentence) and its `launchRunner` sentence.

2. **MEDIUM** — Card 11, `seams.launch` returning an error. The branch is not prescribed.
   - The card says what happens when the runner "has ended or not claimed", but not what happens when `launch` itself returns an error with no pid, such as `cmd.Start` failing.
   - At that point the front holds the key lock, has written the lease and points the key directory at the new run id.
   - The card states no lease expiry, no exit code, no envelope and no way forward for this case, and no test row reaches it. The `helperLaunchEnv=none` row returns a pid, so it does not.
   - Fix: state that a launch error takes the start-failure path (expire the lease, exit 1, same way forward as in finding 1), and add a row for it.
   - Evidence: card 11, Front paragraph (`calls seams.launch`, then the claim-wait sentence); Tests list.

3. **MEDIUM** — Card 13, a `card-verify` runner whose plan read or card match fails. The branch is not prescribed.
   - The runner reads the plan again in `Cwd` and matches the card id, so it can fail where the front succeeded: the plan was unparsable or edited in between, or the card or its Verify line is gone.
   - The card does not say what `exit.json` the runner writes then, or what the front replays.
   - Card 10 only covers "a call that ended under neither": the in-process `test` call's code.
   - Without a stated code, the front may wait out its cap, answer `done: false`, and on the next call join or supersede the same failing run.
   - Fix: the runner writes an unmarked exit with code 1 and the refusal text on `stderr`, naming the way forward the in-process refusal names. Add one row for it.
   - Evidence: card 13's paragraph on how `runDetached` runs a `card-verify` request (it reads the plan again); card 10's Runner verb paragraph on the exit file.

4. **LOW** — Card 13 against card 11's join rule, D6. Editing a card's Verify line may not supersede the run.
   - The join compares HEAD's tree and `verifytree.DirtyPaths` of the worktree.
   - The plan lives under `_lym`, which this hub's geometry keeps in the weft (`PATTERN-durable-vs-ephemeral-state`, the weft-content bullet). An edit to the Verify line may therefore leave the warp's dirty set unchanged.
   - A later `card-verify` call would then join a run of the old line, and "run the same command again" replays the stale line's verdict.
   - I did not confirm this with a tool call. The writer should check whether `DirtyPaths(worktree)` sees `_lym/plan` edits, and otherwise add the card's line text to the key or to `tree.json`.
   - Evidence: card 11's join paragraph (tree and dirty-path compare); card 13's key (`card.ID()` only).

5. **LOW** — Card 1, the negative `keep_warm_max_reruns` refusal names no way forward.
   - The message is `shuttle config: keep_warm_max_reruns must not be negative, got <n>`.
   - Every `shuttleengine.LoadConfig` reads this key, so the refusal also blocks shuttle run starts and `lym shuttle` verbs. Only the resolver degrades to the fallback.
   - Fix: name `lym config shuttle` in the message, as card 2's cap refusal names `lym config claude`.
   - Evidence: card 1's Intent (`LoadConfig` refusal sentence).

6. **LOW** — Card 13, Windows `proc.HoldNamedJob` failure. The branch is not prescribed.
   - The card does not say whether the runner refuses, or logs and runs the line without the job.
   - Without the job, the runner's death leaves the line's tree running, and no front kills it, since `KillTree` is a no-op on Windows.
   - Evidence: card 13's sentence "On Windows the runner first holds a kill-on-close job…"; `internal/proc/tied.go` `KillTree` doc (Windows kills nothing).

7. **LOW** — Card 10, the runner's abandoned line never reaches a caller.
   - The line `gate test: the run was abandoned …; way forward: run the same command again` is appended to `stderr`.
   - Card 11's front never replays a marked exit file, and answers 75 with its own re-run message instead.
   - So the way forward in the line is dead text. It is harmless, but the card prescribes text that nothing reads.
   - Evidence: card 10's Runner verb paragraph; card 11 ("An exit file with the abandoned mark answers as at a passed cap, replaying nothing").

**Refusal list**

| Refusal or stop | Next step it names | Progresses |
|---|---|---|
| Card 2 cap refusal (Prepare, `CheckLoad`) | `lym config claude` | Yes: `CheckLoad`'s before/after compare admits a fixing write |
| Card 10 runner refusals | `lym gate test` / `lym gate card-verify` | Yes |
| Card 11 lock-margin 75 | Run again | Yes |
| Card 11 passed cap 75 | Run again | Yes |
| Card 11 abandoned exit 75 | Run again | Yes: the next call supersedes it |
| Card 11 start failure | Run again | Only for a transient cause (finding 1) |
| Card 11 launch error | Not prescribed | Finding 2 |
| Card 11 replay of a recorded 75 as exit 1 | None needed: a verdict | Yes |
| Card 13 runner plan or card failure | Not prescribed | Finding 3 |
| Card 1 negative-key refusal | None named | Finding 5 |
| Card 3 resolver fallbacks | Not a refusal; the call still runs | Yes |

Tool calls made: none. My context already held the cards and the code they cite.
