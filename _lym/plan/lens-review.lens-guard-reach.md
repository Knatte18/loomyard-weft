```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: whether each guard, refusal, branch and test row the plan adds can be reached from a real entry point, and whether a guarded site recurs unguarded.

1. **BLOCKING — D7, cards 12 and 15: a `Wait` attached within the bound reports a mechanism failure instead of waiting.**
   - Card 15 leaves a record within its bound "to the live relaunch", and card 12 makes that record attachable whatever its pane state.
   - Step 1's `EndStrandProcess` clears the pane id but keeps the strand tracked. The record keeps `Started` true until step 4.
   - So in the attached `Wait`, every liveness tick goes through `checkLivenessTick`'s `!strand.Live && strand.PaneID == ""` branch and returns `errStrandPaneBindingCleared` (`internal/shuttleengine/wait.go`, `checkLivenessTick`).
   - After `maxStatusRetries` such ticks, `Wait` returns a mechanism-failure error.
   - This happens both when the original waiter died during steps 1 to 4 and when it is still relaunching. The run then fails before the bound passes, against D7's criterion "ends the run as a strand that died once the bound passes".
   - No card makes `checkLivenessTick`, or `Wait`'s status-failure exits, read a relaunching record within its bound. Card 15's rows assert only the attach verdict within the bound, never the `Wait` that follows.

2. **MEDIUM — D3, card 5 and Shared Decisions: the "beams pin for a beam worktree" no-op can never fire at creation, so a pinned beam is picked.**
   - `Claude.PickAccount` skips "a beams pin for a beam worktree".
   - At the only creation site, `battenCLI.wire`'s `CreateWorktree` closure, the new beam is not yet marked: its pawl run's Worktree-Create calls `top.Add` without `AddOptions.Beam`, and Beam-Up marks it later. So `hubgeom.EngineWorktree` reads it as a task.
   - The result is that a beam made under a beams pin gets an entry and a count increment. D3 says the pick "does not run for a worktree a pin covers", and the D3 criterion says "a pinned worktree … not picked".
   - The Shared Decisions bullet admits the beam is picked as a task, but records no departure from D3 and no Added-after-Discussion entry.

3. **MEDIUM — D5, card 10: the "listed account with no board directory" state cannot occur.**
   - `creelFlagDirForHook` returns "" only when `c.dirs.Board == ""` (`internal/shuttleengine/claudeengine/claudeengine.go`).
   - The accounts come only from the machine layer, since card 2 refuses them in tracked layers. `configengine.LayerFile` applies the machine layer only when `Anchor` and `Board` are both set.
   - So with no board directory no account is ever listed. The `writeOpenFiles` status-line-only branch is dead, and card 10's `TestOpenLaunch` row "a listed account with no board directory writes settings holding only the status line" cannot be built.

4. **MEDIUM — D7, card 14: a relaunch over a prompt that never went out drops the task prompt.**
   - The switch branch sits "ahead of the existing notice cases", so it runs before `deliverPromptAtHold` (`internal/shuttleengine/hold.go`).
   - That case covers a run whose deferred `PromptLine` never started a turn. It is reachable when the skill-load turn itself ends on the usage limit.
   - Step 2 then always resumes with the continue line and `SkipSkillLoadTurn`, and step 4 writes the launch's empty `PromptLine` over the record's.
   - The resumed session is told to "carry on" with a task it never received, and no card or row guards this state.

5. **MEDIUM — D6, cards 16 and 17: `refusedResume` never sees a resume's first turn when the spec loads skills.**
   - `refusedResume` counts only the "first turn end read past the prompt offset".
   - On the start path, `PromptOffset` is taken after the skill-load turn (`RunState.PromptOffset` doc, `Runner.launch` → `loadSkillsThenPrompt`). So a resumed session's real first turn, the load turn, lies before the offset.
   - An API error there goes through the start path's lasting-API-error hold notice, not through the fresh start.
   - The orch's resume specs carry skills, so card 17's `RefusedResume` misses the same turn.

6. **MEDIUM — D6, card 8: `relaunchForReprompt` throws away its copy fact.**
   - It "ignores the copy fact", yet its resume can copy: `checkResume` now carries the transcript into the account resolved now.
   - Its record's `TranscriptCopied` and `PromptOffset` stay those of the earlier launch. A copied re-prompt resume whose first turn ends on an API error is therefore never read as refused, and card 16's guard is unreachable on this resume path.

7. **MEDIUM — D4, cards 6 and 1: in `OpenLaunch`'s resume line the fresh fallback misses the export.**
   - The export is applied through `accountEnv.wrap`, which uses `sh.WithEnv`. On POSIX that is command-scoped: it covers only the first pipeline.
   - `OpenLaunch`'s `ResumeCmd` is `Chain(exportTTL, OrElse(resumeLine, fresh))`, and `buildOpenCmd`'s doc says the caller exports the environment once in front of the joined lines (`internal/shuttleengine/claudeengine/command.go`).
   - Wrapping the `OrElse` leaves the `fresh` fallback without `CLAUDE_CONFIG_DIR`, so it starts under `~/.claude`.
   - The card says "chains the same statement" but defines only `wrap`; the export there needs `ExportEnv`, as the TTL does. No row pins the fallback's environment.

8. **LOW — D5, card 5: the concurrent-pick guard is underspecified.**
   - The guard compares `Picked` with "the noted time" taken before the lock.
   - If the writer stores its own noted time rather than the time of the write, a second caller that noted a later time while waiting for the lock re-picks, giving two increments.
   - The card does not say which time `Picked` holds, so the "one entry and one increment" row depends on an unstated choice.

9. **LOW — PATTERN change, card 12: `sweepOrphans` keeps a relaunching directory forever.**
   - Card 12 keeps it "whatever its strand" with no bound.
   - The declared PATTERN text keeps it only "until the mark clears or its bound passes". A record past its bound that no attach ever settles is then never swept.

10. **LOW — D7, card 17: the restart closures may run on stale state.**
    - `watchCmd` builds `OrchRestart` "over the state it reads at the call", which is ambiguous between the watch command's start and the closure's call.
    - If it is the start, a second switch restarts from a state naming the old strand GUID and session.
    - The watcher's `tick` already holds the current `st`, and the closures do not take it.

Tool calls:
- Read the code around eight `shuttleengines.New` call sites, and found the function that calls `SessionNamers`, to see which of them launch or wait on runs. The exploration listed only the call lines.
- Named the enclosing functions at `battencli/drive.go:507`, `tendercli/deps.go:383` and `reedcli/watchdog.go:359`. The exploration gave line numbers, not function names.
- Grepped `tendercli` for runner launches and `reedengine/lifecycle.go` for the resume `held` handling. The exploration did not show whether tender launches, or how `planResumeLaunches` holds strands.
