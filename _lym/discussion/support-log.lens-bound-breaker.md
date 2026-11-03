```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker review of the codex-engine decision record and support log.

**Findings**

1. **BLOCKING · D1 and D2 (resume): a live assistant is never found on resume, so it ends with an ended marker even while it is still working.**
   - Counterexample: `ProbeReview` and `ResumeReview` rebuild the table with `reviewSeatValues(&p, reviewFilesIn(e.probeDirOf()))`, which points at `.lym/burler/round-probe`.
   - A fresh `Review` composes into its own `os.MkdirTemp(burlerDir, "review-")` directory.
   - D2 puts the assistant's report and its ended marker in "the half's instruction directory". After a crash, the probe's table therefore names a report path that differs from the one the live assistant writes.
   - Shuttle matches runs by their exact output-file set, so the probe misses the live assistant. Resume then writes the marker under `round-probe`, and the chair's stencil names the old paths. The chair waits on the wrong paths, or goes on without a live assistant.
   - The live assistant's strand is also orphaned: the step no longer tracks it, so nothing stops it.
   - The chair escapes this only because its own output, `ReviewPath`, is stable.
   - Fix: give the assistant's report and marker a stable path derived from `ReviewPath` or the round, not from the temp directory.

2. **MEDIUM · D3 contradicts itself on empty entries.**
   - D3 says "every entry parsed as a model-spec" and also "an empty entry runs no assistant that round".
   - `ValidateModelSpecList` runs `modelspec.Parse` on each entry, so `""` is refused.
   - `ModelSpecList.Unset` reads `[""]` as unset.
   - So `["astra[medium]", ""]` (assist only in round 1) cannot pass validation as written. D3 needs a list grammar of its own that allows empty entries, like the fan list, and must say so.

3. **MEDIUM · D5: the render discards the trust entry Codex itself writes.**
   - In the hook-delivery report, case 5 shows that a prompt in a linked worktree writes its entry for the main checkout's root, not the worktree root.
   - D5 records only worktree roots and renders `config.toml` whole from lym's own record.
   - Counterexample: suppose Codex looks trust up by the repository root. Then lym's worktree-root entry never matches, and the backstop accepts the prompt at every launch. Codex writes the main-checkout entry, and the next Prepare's render drops it again. The prompt then recurs on every launch.
   - The Warn check after startup stays silent throughout, because the worktree root is still present.
   - A Codex accept that writes `config.toml` while another Prepare renders it under lym's lock races too, because Codex ignores the lock.
   - Which key Codex uses for a linked worktree is a fact the design rests on, and D5 leaves it to the backstop. Record the repository's main-checkout root as well, or keep the entries Codex writes.

4. **MEDIUM · D2 bound "its prompt alone keeps it off target files": one hop further, a stray edit gets committed.**
   - Under `workspace-write`, the assistant or its subagents can edit a source file in a Code-Review round.
   - The round's fixer, under `FixScopeSource`, commits in that same worktree, so a stray Codex edit can be swept into a fixer commit or left as a dirty tree for the next gate.
   - The bound names the prompt as the only guard, but not this consequence. Either state it, or have Go check that the worktree has no changes outside the assistant's report when the assistant ends.

5. **MEDIUM · D6 and D7: a Codex crash behind a live pane shell reads as a timeout, not as died.**
   - D7 makes liveness unproven, and D6's resume line is empty.
   - Counterexample: Codex exits or panics and leaves the shell prompt in the pane. The pane is still alive and no `Stop` comes. The run then ends only at `assist_reviewer_timeout_min` (60), and the chair waits that long.
   - D1's bound ("assistant's own timeout plus one await cap") still holds, but D6's claim that a dead pane "ends the run died" does not cover this case, so the record is silent on the common crash shape.

6. **LOW · D6: deny notice and passages in the prompt file.**
   - On Claude these ride the system prompt and survive compaction. On Codex they are user-turn text, which a long fanned review's automatic compaction may summarize away.
   - After that, the guard's notice and the heddle passages may no longer be in context.
   - The rejected alternative (`developer_instructions`) does not state this trade-off.

7. **LOW · D2: the copy "once the chair ends".**
   - Seatengine stops every assistant when the chair ends, so an assistant still writing is stopped mid-write. The partial file it leaves is copied beside the review as if it were a report.
   - The resume path's conclusion must also run the copy. The record names only the end of the chair.

8. **LOW · D9: "a skill is loaded when a tool call of that turn read its path".**
   - Codex reads through shell commands (`cat`, `sed -n`, `rg`), possibly by a relative path or after a `cd`.
   - Matching the absolute path in the command text gives false "missing" results, which trigger a second, wasted load turn.

**Claims attacked**

| Claim | Result |
|---|---|
| D1: stale marker removed on a fresh run | holds, since each Review uses a fresh temp directory |
| D1: wait bounded by the assistant's timeout plus one await cap | holds on the fresh path; broken on resume (F1) |
| D1: the assistant never ends the step | holds |
| D1: hold deferral counts an assistant mid-turn | holds |
| D2: unchanged bytes for a round with no assistant | holds |
| D2: report stays inside the sandbox under `.lym` | holds while the pane's working directory contains the anchor |
| D2: prompt-only write bound | weak (F4) |
| D2: the copy | partial-file gap (F7) |
| D3: empty entry means no assistant | broken (F2) |
| D4: alias fallback and names-equality test | holds |
| D5: no teardown step, check after startup, concurrency | broken (F3) |
| D6: dead pane ends the run died | incomplete (F5) |
| D6: delivery through the prompt file | weak (F6) |
| D7: completion on `Stop`; interrupt read from `turn_aborted` | holds per the spike |
| D8: paste with a separate Enter | holds per the spike |
| D9: classification by path | weak (F8) |
| D10: payload shape | holds; `apply_patch` writes are unguarded, which is acknowledged in spirit |
| D13: Windows refusal before any write | holds |

No further tool calls; my context held everything this lens needed.
