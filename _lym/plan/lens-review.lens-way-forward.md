```yaml
lens: way-forward
outcome: reported
```

Way-forward lens review of the plan's refusal, stop and recovery steps: findings only.

1. **BLOCKING — Card 5 / Card 22 (D5 upgrade hold).** The hold the upgrade writes has no clear path inside this part, so the hub waits forever if this part lands before the adopt part (D6).
   - The upgrade's notice names a re-run of `lym adopt <subjects>`, then `lym tender resume <slug>` once that check is green. Neither step can lift the hold in this part's state.
   - Card 5 says "Clearing a hold is not here: a green check clears it in the adopt part (D6)". Today's `lym adopt` has no hold clear, and `--take-hold` does not exist yet.
   - Card 15 makes `lym tender resume <slug>` refuse a `bolt-hold`-only slug while the hold stands, and Card 13 makes every run on every machine wait.
   - The same gap covers Card 22's "a re-run of `lym adopt` clears it" for a first read that was cut short.
   - Fix: state the part order that guarantees D6's clear lands before or with this part's upgrade, or gate the upgrade's hold write on it.

2. **BLOCKING — Card 10 (`lym machine name` after a copied-file refusal).** The way forward can refuse itself.
   - Card 4's refusal 2 names `lym machine name <new>` for a copied `machine.json`.
   - Card 10's `checkHub` "runs `Registry.Check` for the new name with the file's names, before any write", and only judges whether the id is a copy alongside that.
   - If that Check carries the file's copied id, refusal 2 fires again: the registry holds the id under the other machine's current name, which is neither the new name nor a former or forgotten one. The verb then loops on its own way forward.
   - Fix: Card 10 must run the Check with the fresh id whenever it judges the id a copy.
   - Evidence: Card 10 Intent lines 27–28; Card 4 Check rule 2.

3. **BLOCKING — Card 10 (`nextFile`, D3 criterion "with the refused name among none of this machine's former names").** `dropCurrent` drops `cur.name` only.
   - `nextFile` also appends `cur.hostname` when it is absent (`internal/machinecli/set.go:132`).
   - An unregistered machine named by the hostname rule (the D3 criterion case) has `cur.name == cur.hostname`, so the refused name comes back as a former name.
   - The registration is then refused under rule 3, now naming `--forget-former`, instead of landing.
   - `nextFile` also never reads `Forgotten`, so a name `--forget-former` dropped (Card 11) comes back at the next rename when it is the old name or the hostname. The rule 3 refusal recurs, and its way forward does not stick.
   - Fix: `nextFile` adds no refused or forgotten name.

4. **MEDIUM — Cards 18, 21 and 22 (catch-up failure classes).** A failure other than an unreachable origin or a registration refusal blocks the whole dobby silently, with no way forward.
   - Card 18 joins one run's failed push or hand-over into the error, and the catch-up stays owed.
   - Card 21 classifies `State.CatchUp` only as `origin` or `refused`, posts a notice only for a refusal, and while the catch-up is owed spawns no pass and starts no task for any run.
   - So these leave every run of the worktree held with no notice, no `lym beam status` reason, and no step that progresses:
     - a persistent per-run failure, such as a records push rejected for a reason other than a takeover, or a local pair removal that fails;
     - a Bolt pull that skips `diverged` or `dirty` (`internal/fabricengine/bolt.go:176-180`), which `HubCatchUp` maps to "an error with its detail";
     - a `UpgradeRead` failure (Card 22 keeps the catch-up owed "while it fails").
   - Card 20's `catchUpRun` names no exit and no way forward for these errors either.
   - Fix: classify the remaining errors with their way forward and a notice, or let a per-run failure leave the other runs unblocked.

5. **MEDIUM — Cards 22 and 19 (upgrade intent push meets a takeover).** The upgrade can fail at every start.
   - `RecordPauseIntent` goes through `commitRunIntent`, which calls `PushRun`.
   - When origin's record holds another machine's takeover entry, `PushRun` hands the run over and returns an error wrapping `ErrHandedOver`.
   - Card 19 returns push errors to this caller, and Card 22 treats a step 1 failure as "return before step 3", so the upgrade fails on every dobby start and the prime's catch-up stays owed.
   - `Classify` judges "owned here" from the registry and Bolt owners, not from the run's takeover entry, so this case is reachable.
   - Fix: treat `ErrHandedOver` as "not owned here" and drop the entry.

6. **MEDIUM — Card 15 (`lym tender resume <slug>` refusal).**
   - The refusal says "wait for that adopt's hold to clear, and its entry lifts itself". The entry lifts only through a live dobby's `syncBoltHold` (Card 21).
   - With the worktree's dobby stopped, as after `lym beam stop`, the `bolt-hold` entry stands after the hold clears.
   - `Resume`'s `BoltHold` seam reads the local Bolt with no pull (Card 15: "fills `BoltHold` from `boardengine.ReadOwners` of the hub's local Bolt"), so the refusal keeps naming a hold origin has already cleared.
   - Fix: the refusal names `lym beam start` as the step that lifts the entry, and `Resume` reads a pulled Bolt (Card 14 pulls only for the `Verdict` seam).

7. **MEDIUM — Card 12 (`CoveredError` and `heldLine` for a `bolt-hold` entry).** "wait for that adopt's hold to clear, then <retry>" has the same gap as finding 6.
   - The entry lifts only through a live dobby's `syncBoltHold`.
   - A held process waiting in `WaitUncovered` with no dobby running in its worktree stays held after the hold clears, and nothing in the wording names how the entry goes.

8. **LOW — Cards 4 and 21 (no config directory).**
   - The refusal names setting `XDG_CONFIG_HOME` or `HOME` (or `LOCALAPPDATA`), but the dobby keeps its environment, so its retried catch-up never sees the variable.
   - The way forward also needs `lym beam stop` and `lym beam start` after setting it.
   - D3 says such a machine "runs no dobby", yet Card 8's `EnsureLive` still starts one.

9. **LOW — Card 23 (loom pre-push stop of an owned-elsewhere run).**
   - The stop no longer pauses the run, so `errPushStopped` is the only thing holding it.
   - The card does not say whether the loom driver re-tries the guarded push on each transition, which would post the notice each time.
   - No step states what ends the run's own process before the dobby's catch-up hands it over.

10. **LOW — Card 20 (`reconcileResume` with origin unreachable).** The branch can no longer be reached.
    - The catch-up ahead of it already refuses when origin is unreachable, so "logs and resumes with no marker" runs only for a reconcile that loses origin between the two calls.
    - The card states the branch as the marker's replacement without saying so.

Tool calls:
- Read `internal/fabricengine/bolt.go:255-300` and grepped the `BoltSkip` constants. My context lacked how `Bolt.Pull` answers `SkipGit` and which skips it returns, which `HubCatchUp`'s error mapping and finding 4 depend on.
