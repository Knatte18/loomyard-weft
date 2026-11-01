# Card 11 — Adopt holds the runs it takes over

**Create:**
- `plan:internal/adoptcli#holdsFileName` -> `const holdsFileName = "holds.json"`
- `plan:internal/adoptcli#triesFileName` -> `const triesFileName = "fix-tries.json"`
- `plan:internal/adoptcli#readHolds` -> `func readHolds(dir string) ([]string, error)`
- `plan:internal/adoptcli#appendHolds` -> `func appendHolds(dir string, slugs []string) error`
- `plan:internal/adoptcli#removeHolds` -> `func removeHolds(dir string, slugs []string) error`
- `plan:internal/adoptcli#readTries` -> `func readTries(dir string) (int, error)`
- `plan:internal/adoptcli#writeTries` -> `func writeTries(dir string, count int) error`
- `plan:internal/adoptcli#holdEntries` -> `func holdEntries(record *pauserecord.Record, slugs []string, now time.Time) (held []string, paused map[string]string)`
- `plan:internal/adoptcli#holdTargets` -> `func holdTargets(deps adoptDeps, here string, reconciled boardengine.Owners, tasks []boardengine.Task, runs []string, writing []string, rootGoing bool, outcomes map[string]*slugOutcome) (map[string][]string, error)`
- `plan:internal/adoptcli#reconcileWarnings` -> `func reconcileWarnings(report fabricengine.OriginReport, adopted []string, outcomes map[string]*slugOutcome, req request) []string`
- `plan:internal/adoptcli#TestHoldStore` -> `func TestHoldStore(t *testing.T)`
- `plan:internal/adoptcli#TestHoldEntries` -> `func TestHoldEntries(t *testing.T)`

**Delete:**
- `internal/adoptcli#resumeHaltedChildren`
- `internal/adoptcli#adoptedUnder`

**Edit:**
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#adopt`
- `internal/adoptcli#outcome`
- `internal/adoptcli#slugOutcome`
- `internal/adoptcli#resumeSlug`
- `internal/adoptcli#liftOwnerPauses`
- `internal/adoptcli#heldBeamWayForward`
- `internal/adoptcli#ownerWriteError`
- `internal/adoptcli#runBeamSteps`
- `internal/beamcli#createCmd`
- `internal/battencli#OwnedElsewhereError.Error`
- `internal/adoptcli#adoptFields`
- `internal/adoptcli#Command`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#world`
- `internal/adoptcli#newWorld`
- `internal/adoptcli#world.deps`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli#TestAdoptFields`
- `internal/adoptcli_test#TestAdoptIntegration_SecondCloneTakesOver`
- `internal/adoptcli/doc.go`
- `internal/beamcli/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/pauserecord#HoldCaller`
- `plan:internal/adoptcli#subjectDir`
- `internal/adoptcli#movedRuns`
- `internal/adoptcli#projectOf`
- `internal/adoptcli_test#holdOwnerPause`
- `internal/adoptcli#slugOf`

**Intent:**
D4's hold, D1's step order, D2's warning and the hand adopt's try reset (D5, D6); the resumes leave adopt, and card 12 puts them at the check's green end.

Stores, beside the check record in the subject's directory (`subjectDir`), each read and rewritten through `internal/state` under its own lock and never touched by the check record's writes: `holds.json` holds the held slugs, `appendHolds` adding the slugs not listed and `removeHolds` dropping the named ones in one update, so a slug a second adopt appends meanwhile stays; `fix-tries.json` holds the fix-try count, `readTries` reading an absent file as 0.
An absent file reads empty; an unknown key reads (PATTERN-tolerant-records).

The hold, written after the pairs are added and before the owner write:
- `adopt` reads the owners again right after the reconcile, through `ReadOwners`, so the owner a hold is decided by is the Bolt the reconcile left (PATTERN-owner-machine); the earlier read still names the entries.
- `holdTargets` returns, per subject, the slugs to hold: each moved run that went through `takeOverSlug` with no stop, error or warning, whose effective owner in the reconciled owners is another machine, and whose effective owner after this call's owner write will be this machine: its project is a beam being written or `root` while the prime is being written, and for an adopted beam's own pawl run its parent is owned here or is being written; and each task entry under a subject being written (its `ParentOf` the beam, or the prime for `root`) that is a task, not done, released with an empty status and not `Front`.
  A run whose pair `LivePair` reads live here is not held, and reports running here; a run that will still read owned elsewhere, such as a nested beam keeping its own owner, is not held.
- `appendHolds` lists them in each subject's holds file, then one `pauserecord.Update` through the new `UpdatePause` seam applies `holdEntries`: a slug whose own scope holds an `OwnerCaller` entry has that entry's caller turned into `HoldCaller` in place, keeping its pause time; one already holding a `HoldCaller` entry keeps it; one holding an entry of any other caller keeps it, gets no hold and is returned in `paused` with that caller; any other gets a new `HoldCaller` entry of its scope at now.
  The hold is written straight through the record, never through `tenderengine.Pause`, so it waits no grace and sends no Esc.
- A slug in `paused` is dropped from the holds file again by `removeHolds`, so the green end never lifts what adopt did not hold.
- An owner write that fails, or that another machine's write outruns, leaves the holds standing; the envelope still carries each held slug, so a re-run finds them in place.

After the owner write: `resumeSlug` no longer resumes; a held slug's outcome reads `Run` `held` with `Held` true and `PauseStanding` `held for the adopt check of <subject>: it starts by itself once that check is green; a re-run of lym adopt <subject> checks again`, never naming `lym tender resume`; `liftOwnerPauses` runs only for a slug adopt did not hold; a slug with another caller's entry keeps its standing line naming that caller.
`resumeHaltedChildren` and `adoptedUnder` go, with the `ResumeRun`, `ChildStatus` and `ResumeChild` seams of `adoptDeps`, which card 12's check takes over; `resumeChild` and `readStatus` in `deps.go` stay for it.
New seams `Tasks` (the board's entries, `boardengine.ReadTasks`), `UpdatePause`, `Holds` (read, append and remove over a subject's holds file) and `ResetTries` are filled by `hubDepsAs` over the hub.

Texts that say adopt resumes: `heldBeamWayForward`'s parent takeover and `runBeamSteps`' doc comment that mirrors it, `ownerWriteError`'s two errors, inside `adopt` the lost-owner error and the read-back error, `OwnedElsewhereError.Error`'s standard refusal, and `adoptcli/doc.go`'s nested-beam bound and landing-hold paragraph each name the subject's adopt check, which resumes the runs at its green end, in place of adopt resuming them or nothing being resumed; `beamcli/doc.go`'s line and `createCmd`'s long help line that only the create verb, a hand pass or `lym adopt` starts or resumes a beam's run name the adopt check's green end in place of `lym adopt`.
`adopt.go`'s file header names the hold before the owner write and no lift or resume; `deps.go`'s header drops the run and child resume seams.

Checks: adopt starts the check of each subject it recorded an owner for and of each named subject, `root` included when named, whose holds file lists a slug a live `HoldCaller` entry of its own scope still covers, so a re-run after a failure starts a new check even when no owner changes; before each start it writes the subject's try count to 0 through `ResetTries` (D5: a hand adopt resets it; a check started otherwise does not).

D2: `reconcileWarnings` returns, for each branch the reconcile report holds whose outcome reads `Prime`, or whose `Slug` is an adopted beam, in state `rewritten` or `diverged`, or `ahead` with `Moved` unset, that no stop of that slug already reported and that `sharedStopText` did not refuse on, `<branch> is <state> against origin and was left unmoved: run lym fabric pull in <worktree>, then re-run lym adopt with the same arguments`, with the branch outcome's own `WayForward`, when the reconcile set one, following as detail in parentheses after the worktree, so every line names `lym fabric pull` as D2 requires.
The worktree named after `lym fabric pull in` is the outcome's `CodeWorktree` for a records branch, where its fabric verbs run, and its `Worktree` for a code branch.
A records branch the reconcile replayed reads `ahead` with `Moved` set, its replayed commits sitting on origin's head, so it gets no warning.
Adopt goes on and `outcome.Warnings` carries them as the envelope's `warnings`.
Adopt runs no `lym fabric pull`.

`Command`'s help lists the steps in D1's order: reconcile, the local-commit stop, the pairs, the holds, the owner write, the beam steps, one check per subject; and that red no longer only reports.

Tests:
- `TestHoldStore` over a temp dir: append twice keeps one of each slug; remove drops only the named; tries read 0 when absent and the written count otherwise; a file holding an unknown key reads.
- `TestHoldEntries`: an owner entry turns into a hold with its pause time kept, leaving one live entry of the scope; an existing hold stays one; an operator's entry stays and the slug is returned paused with its caller; an uncovered slug gets a new hold.
- `TestAdopt`: the `world` gains tasks, the pause updates and the holds and tries files per subject, an `afterReconcile func(*boardengine.Owners)` hook its `Reconcile` runs on `w.owners`, as `race` runs after the owner write, and `order` entries `holds:<subject>` for a holds append, `pause` for the pause update and `owners` for the owner write; rows: a moved run under a beam owned elsewhere gets a hold listed in the beam's holds file, `holds:<beam>` and `pause` coming before `owners` in `order`; a beam the pre-reconcile `w.owners` names as this machine's and `afterReconcile` turns to another's gets its runs held; a released task under the adopted beam is held and a released front task is not; a run with an owner entry ends with one live entry, of `HoldCaller`; a run with an operator's entry keeps it, is not held and its standing line names the caller; an adopt with `--keep-local` over a live pair under a beam owned elsewhere leaves that run unheld and reports it running here, and a nested beam owned elsewhere is not held; a refused owner write leaves the holds and lists them; no run is resumed; a second adopt right after changes no owner and reports the same held slugs; a re-run with holds standing and no owner change starts the subject's check and resets its tries; a rewritten branch of the prime with no `WayForward` is a warning naming `lym fabric pull`; a diverged branch of an adopted beam's pair carrying a `WayForward` is a warning naming `lym fabric pull`, with that way forward following it; an ahead branch of the prime with `Moved` unset is a warning; a records branch of an adopted beam's pair the reconcile left unmoved in state diverged, its `Worktree` the records sibling and its `CodeWorktree` the pair's code worktree, is a warning naming `lym fabric pull` in the `CodeWorktree` path and not in the `Worktree` path; a records branch of the prime the reconcile replayed, reading ahead with `Moved` set, gets no warning; a branch of a run the adopt stopped on gets no second warning; a shared Bolt stop refuses as before with no warning; a branch of a beam not adopted gets none.
- `TestAdoptFields` asserts `warnings`.
- `TestAdoptIntegration_SecondCloneTakesOver`: the second clone's adopt holds the task, its owner pause turned into a `HoldCaller` entry, and resumes nothing; with the first clone's commit pushed to the prime's code branch and to the task's pair branch before the second clone adopts, the second clone's prime and task pair branches read origin's tip after the adopt, through the reconcile step alone, D2's criterion that adopt runs no `lym fabric pull`.

`adoptcli/doc.go` replaces steps 4 to 6 with the hold, the owner write, the beam steps and the per-subject checks, and states the holds and tries files.
`docs/overview.md`'s adopt paragraph replaces the owner-pause lift and the resumes with the holds, which the subject's green check lifts.

**ImpactSummary:** adopt's sequence gains the hold before the owner write and loses its resumes; its envelope gains held slugs and warnings; inside adoptcli and its tests, plus the texts in beamcli's doc and create help and battencli's owned-elsewhere refusal that said adopt resumes.

**Commit:** 11: adopt holds the runs it takes over until their subject's check
