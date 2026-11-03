```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the merge range, taking each changed function or branch with a concrete case. I read from inherited context only.

**List: function or branch, case run, correct or not**

- `skipsCheck`
  - Pushes off, `SkipGit` or `SkipPush`: returns true with no further reads. Correct.
  - `isPrime` fails: returns `(false, err)`. `guard` then warns and lets the push go. Correct.
  - A pause covers the run: returns true. Correct.
  - Empty `currentProducer`, meaning no status file: not Finalize, so the check runs. Correct.
  - `currentProducer` read fails: returns `(false, err)` and the push goes unchecked, as the doc says. Correct.
- `prePushCheck.guard`
  - Unreachable origin and failed reconcile: both return nil with a Warn. Correct.
  - `EntryOwnedElsewhere` with an empty `AdoptSubject`: names `lym adopt <slug>`, pauses under the owner, posts, returns an error wrapping `errPushStopped`. Correct.
  - `EntryEnded` with the `landed` read failing: no stop, the push goes. Correct.
  - `EntryEnded` with the parent already holding the work: the push goes. Correct.
  - `EntryStopped` with no named branch: falls back to "a branch every run shares". Correct.
  - The pause call fails: the failure text is added to the notice and the stop still returns. Correct.
  - Active entry with moved branches: posts one notice and returns nil. Correct.
- `announcedBefore`
  - Same key already stored: returns true, so no repeat notice. Correct.
  - Different key, or file absent: rewrites the key and returns false. Correct.
  - Unreadable or unwritable record: returns false, so a notice may repeat instead of being lost. Correct.
- `guardedCommit`, `guardedSpawn` and `guardedPush`
  - A stopped commit returns `(false, err)`, and `newCommitStatusSeam` turns `errPushStopped` into `(false, nil)`. Correct.
  - A stopped spawn returns nil. Correct, since the commit that precedes it already passed the guard.
  - A stopped push returns the error to the caller. Correct.
  - After a stop, the next transition finds the pause covering the run. `skip` is then true, so the commit proceeds and the push waits at the network gate, as the doc says. Correct.
- `commitStatusDepsGuardedBy` and `loomCommitStatusDeps`
  - Building the check does no I/O, so the `wireLightweight` comment's "no I/O at build time" claim holds. Correct.
  - Every beam field in `wire()` survives in the merged `wiring.go`. The range diff touches only the pre-push hunks. The beam's `DefaultEngine`, `assistBounds` and `Segments` hunks are intact. Correct.
- `statusWatchLine`, `PrintStatusLinesOnChange` and `waitingNote`
  - Unreadable status file: the line and the key are both the unavailable line. Correct.
  - A wait note whose text differs but whose steady form is the same: suppressed. Correct.
  - A changed wait target: prints. Correct.
  - A settled run, or a nil hook: returns the zero `WaitNote`. Correct.
- `gateWaiting`, `verifyWaiting`, `shuttleWaiting`, `parentLockWaiting` and `timedNote`
  - Every branch returns both text and steady form.
  - Joins are consistent between the two forms. Correct.
  - Two gate waits plus a `next` note: text and steady are joined in the same order. Correct.
- `checklistPoll` and `reviewWaitingFor`
  - A nil location returns a nil hook, and `waiting != nil` guards it. Correct.
  - `arm.go:439` and the checklist call site both compile against the `WaitNote` signature, since vet is clean.
- `createSteps` (beamcli)
  - Run found, done, pair gone: `ClearMotifRun` runs, and the later `state != Done` test is false, so the else branch starts a fresh run. Correct.
  - `ClearMotifRun` fails: the error is returned with a rerun hint. Correct.
  - Run done and pair present: nothing cleared, and the start refuses as before. Correct.
- `drivable` and `planPass` (dobby)
  - Paused with `ReachedParent`, covered, no lift: `passNone`. Correct.
  - Paused with `ReachedParent`, covered, lift: full pass. Correct.
  - Paused with `ReachedParent`, not covered: full pass, never a continue pass. Correct.
  - Paused without `ReachedParent`: `passNone`. Correct.
  - `drivePasses` clears `liftPending` for a run that is not drivable. Correct.
- `taskWorktreePresent` and `taskWorktreeLocator` (battencli)
  - A stray directory that does not resolve as a worktree now reads as absent. Correct.
  - A stat error other than not-exist is still returned. Correct.
  - The locator's later calls stat the path and refuse an absent one by name. Correct.
  - Weak spot, not a finding: the locator's retry path stats only, so a path that stays but stops resolving reads as present after the first success. This is main's intended "present pair never changes" rule.
- `WriteChildRecipe` after the child seed (battencli `wire.go`)
  - A write failure returns the error from the seed seam after the child was seeded. This is a rerun-safe failure and is main's own design. Not flagged.
- `finishGitWorktreeRemoval`
  - Clean exit, target absent: returns the nil `gitErr`. Correct.
  - Clean exit, target refilled: removed through `removeWorktreeLeftover`, and the leftover error is wrapped with the "stop whatever refills it" text. Correct.
  - Git failure that is not a `GitError`: returns `gitErr`. Correct.
  - Git failure with the worktree still registered, or the worktree list unreadable: returns `gitErr`. Correct, since the registration proof is missing.
  - Nil `seam.refill`: skipped. Correct.
  - `removeLocalWeft` and `removeWeftWorktree`: the new seam parameter is threaded through both callers. Correct.
- `chooseResumeOnWake` (hibernatecli)
  - `--resume-on-wake=false` given: returns false without reading config. Correct.
  - Flag absent: reads `hibernate.yaml` through `LoadOrTemplate`. An absent base file resolves the template, so the default is false. Correct.
  - Config load fails: the request is refused. Correct.
  - `ConfigDirs` now carries `Records`, and no layer reads it, so `hibernateconfig.LoadConfig` is unaffected.
- `battenRunCommitPaths` and `ParentNoticeFile`
  - The child-recipe path is added only when the file exists. Correct.
  - The notice file sits under `ScratchDir`, ephemeral, and is not staged by `statusCommitPathspec`. Correct.

**Findings:** none. No wrong result, swallowed error or inconsistent failure-path state in the merged range. Main's logic reproduces main's patch, and the beam's fields in `wiring.go` are intact.

**Tool calls:** none. The manifest context covered every function above.
