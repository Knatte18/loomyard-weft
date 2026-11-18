```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every decision, acceptance criterion and card branch for a carrying card and a test row that would fail if it broke.

**BLOCKING**
None found.

**MEDIUM**

1. **D3, card 5, Shared Decisions bullet 6.** A new beam is picked as a task worktree even under a beams pin, against D3's rule that no pick runs for a worktree a pin covers.
   - D3 says "It does not run for a worktree a pin covers". Card 5's no-op is only "under a beams pin for a beam worktree".
   - The overview admits a new beam is "picked as a task worktree" because Beam-Up marks it after Worktree-Create.
   - Under a beams pin, each new beam therefore writes an entry and increments the counter, which skews D3's shares.
   - No row covers beam creation, or creation under a beams pin. That leaves the D3 criterion "a new beam's creation … each pick; a pinned worktree … not picked" without a row.

2. **D8, card 7, Shared Decisions bullet 5.** The plan departs from a decision with no `Added after Discussion` entry.
   - D8 says the live-transcript readers read "the run's launched directory (D4), not the worktree's current resolution". A run with no record reads "under the worktree's resolution, then the other known directories".
   - Card 7 reads the newest copy among all known directories instead.
   - The rubric's fidelity rule says a departure from a Decision needs the parent to record it. A Shared Decisions note does not license it.

3. **D4, card 6.** The criterion "exporting `CLAUDE_CONFIG_DIR` … after any `env` module export" is argued, not tested.
   - The card relies on reed's `paneEnvExports` running ahead of the command.
   - `TestPrepare_ExportsTheResolvedAccount` and the `TestBuildLaunchCmd` rows only assert that the export or unset is on the line. No row puts an `env`-module `CLAUDE_CONFIG_DIR` beside it and checks which value wins.

4. **D4 and D9, card 3.** `tendercli`'s `newShuttlePair` is not switched to `NewForWorktree`.
   - It builds a launching `NewRunner` from `shuttleengines.New(shuttleCfg, configDirs)` (`internal/tendercli/deps.go:383-387`). With a zero worktree, its sessions resolve only the hub pin and ignore the worktree's entry.
   - I found no direct `runner.Start` in tendercli, so whether it launches needs confirming. `weavercli/wiring.go:40` (`resetDepsFor`) is the same shape but looks reset-only.

5. **D9.** The criterion "an orch and a dobby in the prime under a beams pin run under the pinned account" has no row. No test reaches the orch's or the dobby's launch with a prime-kind worktree under a beams pin, through `wireCLI` or `primeShuttle`.

6. **D6, card 16.** No row covers the refused resume on a spec without `APIErrorRetries`.
   - `retryAPIError` returns at `run.spec.APIErrorRetries <= 0` before counting (`apierrorretry.go`). The card does not say the refused-resume case runs ahead of that return.
   - Its rows only differ in `TranscriptCopied`. A copied resume on a spec with no retries would never start fresh, and no row would fail.

7. **D7 criterion 1, card 14.** The real-engine half is unreached.
   - "After the relaunch the launched-account record, the settings' status line name and reed's stored resume command all name the new account, the resume is not refused for a live holder" is tested only on `switchFakeEngine`.
   - No row runs the claude engine's `Prepare`, `CheckResume` and `SwitchAccount` together to show `account.json`, `statusLine.command` and `ResumeCmd` name the new account.

8. **D7, card 14 branches.**
   - "Any other refusal [than `ErrNoTranscript`] … ends the run as a strand that died" has no row.
   - A record already terminal at the mark ("reports stopped and the relaunch ends with the stop's outcome") has no row.
   - "The run ends once, with one session-end usage record" has no row asserting a single session-end report across the relaunch.

9. **D8, card 7 branches.** Four have no row:
   - A `claude.accounts` that fails to load is logged and the remaining directories are read.
   - Outside a hub, the process's own `CLAUDE_CONFIG_DIR` and `~/.claude` are read.
   - `installedPlugins` and `sessionScopeFor` take the launch's effective directory.
   - `tokencount`'s `main` resolves its worktree through `lymcwd`.

   The criterion "`tools/tokencount` and the session-end usage record count a session run on a listed non-default account" is argued from the `TestRecordReaders` row ("which … count"). Neither consumer is exercised.

10. **D7 verb, card 9.** `lym claude account show` has no content row. `ShowAccounts` (resolved account, pins, each account's weight, windows, `blocked_until`, headroom) is untested; `TestCommand` checks only verbs and argument counts.

11. **D3 and D2, card 5 branches.** Two have no row:
    - "A failed `Names` call refuses the pick and writes nothing."
    - "Every pick failure is logged at Warn and never fails the create or the adopt": `TestWire_CreateWorktreePicksTheNewWorktreesAccount` covers only a failed add, not a failing pick.

12. **D1.** "With fewer than two accounts of positive weight … no switch runs; a pin still applies."
    - Card 13's "fewer than two … returns false" has no row in `TestClaude_SwitchAccount`.
    - The criterion "a pin naming it is still exported" with one account has no row in card 6.

**LOW**

13. **Card 8.** "A carry failure refuses the resume" has no row in `TestRunnerStart_Resume_CarriesTheTranscriptFirst`.

14. **Card 11.** Three branches are reached only through the fake in card 14:
    - `ReleaseStrandHold` itself.
    - `EndStrandProcess` failing after the mark.
    - `RelaunchStrand` on a tracked strand that is not held.

    `TestEndAndRelaunchStrand_KeepTheGUID` covers only the untracked refusal.

15. **Card 10.** The row "a listed account with no board directory writes settings holding only the status line" looks unconstructible.
    - `creelFlagDirForHook` is empty only when `dirs.Board == ""`.
    - Without a board, `LayerFile` applies no machine layer, so no account can be listed.

    The D5 case "an `OpenLaunch` session with no creel flag directory" may be unreachable in production. The row as described cannot reach it through real config.

16. **Card 3.** "A failed read logged at Warn and taken as task" can read the prime as a task, and card 5 would then pick it, breaking D9's "the prime is never picked". No row covers it.

17. **Card 12.** `ReadAgentWaits` is edited with no row.

18. **Card 15.** A within-bound attach is read as live and attachable by card 12's disposition. No row covers an attach Wait running beside a live relaunch in another process, which is two Waits over one run. The guard-reach lens may own this.

Tool calls beyond my context:
- Grepped the `shuttleengines.New` callers in `weavercli`, `tendercli`, `loomcli` and `battencli`, plus the `SessionNamers` call site, to check which launching wirings card 3 misses and that `enterSession` is the namer's caller. The exploration listed the callers without saying which launch.
- Found the function enclosing `watchdog.go:359`, and grepped `tendercli` for `runner.Start`/`Run` calls, to confirm card 7's `enterSession` target and whether tender launches. The context had only the line number.
