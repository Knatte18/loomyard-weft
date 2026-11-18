```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. I checked each refusal, stop, reset and recovery step the cards add, and whether its next step can succeed from the state it leaves.

1. **MEDIUM** — Card 17 (D7 orch half): when the `Resume` or `Fresh` restart fails, its fallback cannot work.
   - Card 17 says a failed restart is logged at Warn and the watcher waits the limit out as today.
   - But `relaunchOrch` has already withdrawn the orch's mail and removed the old orch strand, which ends the limited process, before `launchOrch` can fail.
   - `tickUsageLimit` (`internal/orchengine/usagelimit.go`) then renews a hold on a strand that is gone and injects the wake line into it. The orch never comes back, and no step names a retry or `lym orch start`.
   - Fix: the failure path should retry the restart on a later tick, or record `FreshLaunchFailure` as `startOrch`'s `saveFreshLaunch` path does so `retryFreshLaunch` picks it up. Otherwise it should say the operator runs `lym orch start`.

2. **MEDIUM** — Card 14 (D7): `switchAccountAtLimit` does not say what happens when `AccountSwitcher.SwitchAccount` returns an error.
   - Card 13 can return one: a failed `recordBlocked` or `updateAccountFile` write, or an unreadable `account.json`.
   - Card 14 handles only true and false. Card 17 states that an error is logged and the limit is waited out; card 14 does not.
   - Fix: say that an error is logged and the limit falls through to today's cases.

3. **MEDIUM** — Card 6 (D2, D4): any failure to read the account file refuses every launch, and the refusal names no way forward.
   - D2 calls `accounts.json` a cache that a fresh or wiped copy reads as empty. Yet card 6 says "A config or account-file read failure refuses the launch".
   - A truncated or undecodable `accounts.json` or `usage/<account>.json` gets `state.ReadJSON`'s `ErrDecode`. Every Claude launch in the hub then refuses, and nothing tells the operator that deleting the file is safe.
   - `PATTERN-refusal-way-forward` asks for a refusal that names its way forward and a test that reaches it.
   - Fix: name the file and that removing it is safe, with a test row. Or read an undecodable cache as empty and log it.

4. **MEDIUM** — Card 12 (D7 orphan sweep): `sweepOrphans` keeps a relaunching record "whatever its strand", and nothing ends a record past its bound whose strand is gone.
   - Card 15 settles such a record only through an attach's `Wait`. Its `dispositionCandidate` change covers only "a tracked strand", and an untracked one goes "as today" to `leftoverThenAgeVerdict`, which returns respawn-eligible.
   - `removeSupersededStrands` stops only a tracked, removable strand (`internal/shuttleengine#Runner.removeSupersededStrands`), so the record is never made terminal.
   - The sweep then keeps the directory forever. Past its bound, `unfinished` reads the record as unfinished, so every sibling `Start` carries its instructions over again.
   - Fix: keep the directory only while the record is within its bound. Or have the respawn-eligible branch end a past-bound relaunching record.

5. **MEDIUM** — Card 8 (D6): `copyTranscript` is not told to create the target project directory.
   - The resolved account may never have run in this worktree, for example after `lym claude account set` or a first switch. Its `<dir>/projects/<encoded-workdir>/` then does not exist.
   - Card 8 says a carry failure "refuses the resume like the check's own refusal", and that error is not `ErrNoTranscript`.
   - So `launchOrch` fails the start rather than launching fresh (`internal/orchcli#orchCLI.launchOrch`), and card 14 step 3 ends the run as died.
   - The card's test rows don't say whether the fixture pre-creates the directory, so none reaches this case.
   - Fix: the copy creates the project directory, with a row where the resolved account has no project directory yet.

6. **LOW** — Card 2 (D1): the refusal of an account key in a tracked layer names only `lym config claude --layer machine`, which does not clear it.
   - The refusal stands "also when the machine layer holds the key", so setting the key in the machine layer leaves the hub or worktree file as it is, and the refusal repeats.
   - `loadConfigOrRefuse` (`internal/shuttleengine/claudeengine#loadConfigOrRefuse`) also appends "with lym config claude --layer <default|worktree|local>", which never names the machine layer.
   - Fix: the refusal names removing the key from the named tracked file, for example `lym config claude --layer default|worktree`, and the machine layer as where it belongs. A test should reach that text.

7. **LOW** — Card 11 and card 14 step 1: after a partial `EndStrandProcess` failure, the release path can leave the run unrecoverable.
   - `EndStrandProcess` marks the strand held, then kills and reaps. A reap failure can come after the pane is already killed.
   - Card 14's failure path calls `ReleaseStrandHold`, clears the mark and says the limit is "handled as today". The pane is dead, so `Wait`'s next liveness tick ends the run `OutcomeDied`, and reed's resume relaunches the strand with the old account's `ResumeCmd`.
   - Fix: say that the release path applies only when the pane is still live, and that it otherwise goes on with step 2.

8. **LOW** — Card 14, card 16 (D6): after a fresh relaunch, the step 4 write does not clearly reset `TranscriptCopied`.
   - Step 4 writes "`TranscriptCopied` the check's report", but the fresh fallback runs no check.
   - If the copied value stays true, the fresh session's first API-error turn end is again a refused resume (card 16's `refusedResume`), and that loops into fresh relaunches.
   - Fix: say a fresh launch writes false.

9. **LOW** — Card 9 (D7 verb): outside a hub, `accountKeeperHere` names `lym config claude --layer machine` as the way forward.
   - That command has no machine layer outside a hub, where `configengine.LayerFile` needs both `Anchor` and `Board`. The step cannot succeed from there.
   - Fix: the way forward should be running the verb from a worktree of a hub.

I made no further tool calls.
