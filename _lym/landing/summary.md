# Push a transition's records before the run acts; hand over on takeover

A run's state is now on origin before the run acts on it, and another machine's takeover entry in the run's own records hands the run over instead of being replayed over.
The operator requires the whole state on GitHub at all times.
Before this change a transition's push could lag or fail with nothing waiting for it, and a rejected push replayed the records over whatever origin held.

## What changes

### Synchronous transition push

- A shed transition takes a `pauserecord.Admit` admission, commits its records naming the code tip, and pushes records side first and code side after.
  The next producer starts only once every side has landed.
  A pause or hold written after the commit stops the next producer, never that push's first attempt.
- `fabricengine.PushPairAnchored` and `coalescePushSides` send the records side first, and the code side only once the records side has landed.
  A failed or rejected records side withholds the code side, and the caller's next pair push sends it.
- `lym tender pause`, `lym tender resume`, `lym batten cancel`, the pull-request record and the fixer attempt counts use the same synchronous push.
  With origin unreachable the three verbs act here at once and owe their push; `lym batten cancel` then names a re-run once origin answers.
- Bolt writes that carry a decision push synchronously: a run claim, a landing's board `done` and an owner.
  A board write that only displays a run status stays on the detached board sync.
- Retired: the per-transition detached push in `statuscommit`, the teardown-lock respawn, and the pending-flag respawn (`respawnForeignFlags`).
  A transition's push that meets a held teardown lock waits up to `fabricengine.StatusPushLockWait`, then turns owed.
  The anchor change's push and tender's prime push stay detached, since neither carries a run transition.

### Owed push

- A push that fails for a transient reason leaves the run waiting at its transition and retries on a backoff behind the network gate.
- `pairteardown.ReadRun` reads the owed push into `runfsm.Inputs.PushOwed` (renamed from `PushOwedOffline`), with its last error.
  The error reaches the verdict, the refusal and `lym beam status`.
  The refusal names the owed push and its last error, and its way forward is a re-run once the push lands.

### Takeover hand-over

- A rejected records push fetches the branch and reads the run's own record from origin's tip before any replay.
  The caller hands `fabricengine` the set of names that read as this machine; the registry part can pass its names in later without a signature change.
- On a takeover entry naming another machine, and not named by a hand-over mark here, the run hands over:
  it pushes no code side, ends its processes for the slug through `pairteardown.EndSubjectProcesses`,
  archives local tips (and the local parent merge) under tags, posts one notice, writes the hand-over mark under the hub's `.lym`,
  and removes the local pair through `pairteardown`'s local mode when the caller runs outside the pair.
- A rejection whose record holds no takeover entry, such as one over other runs' commits on a shared records branch, replays as before.
- `Finalize` reads the owner from a fetched Bolt before its merge and before each later push.
  It pushes the parent branch, then its transition, then writes the board `done`; before, `done` came first.
  A takeover between the local merge and the parent push leaves origin without the work; the adopter merges again and the old owner resets its local parent branch.

### Code-tip wait

- `ReadRun` sets `CodeTipMissing` while origin's code branch lacks the tip the run's last transition names, and the act rule answers `wait`.
- After `runfsm.CodeTipGrace` (30 minutes) the owner's dobby gives the tip up with a records commit returning the run to its previous transition.
- A machine owing a tip pushes it with a lease and re-reads the records afterwards; if the transition no longer names the tip, it resets origin's code branch with a lease.
  The owner's rejected code-side push resets origin's code branch the same way when origin's tip is neither the last transition's tip nor an ancestor of it.
- `lym beam status` names the missing tip and the machine that owes it.

## Design decisions

- The takeover entry is a per-run compare-and-swap on records branches many runs share, and costs no extra fetch per transition.
- A transition commit carries a `Run-Transition` trailer (`fabricengine.TransitionTrailerKey`).
  The code-tip readers select the newest commit whose trailer names the slug, so other `Warp-SHA` commits from `lym fabric` do not count.
- The hub stop (`lym beam stop --all`) writes no per-run pause or cancel intent.
  A pushed intent would read as paused on every other machine.
- Only the printed `parent_beam` names the prime `root`; code reads through `boardengine.ParentBeam` and `hubgeom.ParentOf` keep the empty slug.
- New pattern `PATTERN-run-fsm`; `PATTERN-pair-remote` and `PATTERN-daemon-wakeups` are updated.

## To check by hand

- The plan's code-tip read from every `Warp-SHA` commit (the `Run-Transition` trailer fix) and the owed-tip hook for the give-up test were open at plan review.
  Check `lastTransitionCodeTip`, `OriginCodeTip` and `ReturnToPreviousTransition` in `internal/fabricengine`, and the `SetOwedTipPushedHookForTest` row in `TestHandOverTips`.
- `owed-code-tip-not-pushed-when-origin-lacks-code-branch`: check that an owed code tip is pushed when origin has no code branch yet.
- `pushbeam-takeover-read-off`: check that the beam push in `internal/beamcli` reads origin for a takeover when its push is rejected.
- `recorded-landing-commit-archived-merge`: check that `internal/landingshed` records the landing commit again after the parent merge is archived.
- Only the tier0 and tier1 gates ran per batch; tier2 and tier3 did not run, so run those suites before landing.
