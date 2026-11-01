# Plan-Burler-Fix round 1 — fixer report

## Changes

- F1 (BLOCKING), fixed differently: card 4 creates `endOnlyError(slug, remove, report)`, which `Cancel` returns when `cancelRun` returns no error.
  It is nil for a full cancel and, for an `EndOnly` report, an error naming origin unreachable and `cancelCommand(slug, remove)` as the way forward once origin answers, not matching `ErrNothingLeft`.
  The verb keeps its end-only report and exit.
  `TestCancelRun`'s end-only rows assert that error and that the entry's run status and the run record stay; a full-cancel row asserts nil.
  Card 6's step 1 and its `TestRemove` row name the end-only error as a cancel error that fails the step.
  The proposal's test row could not be carried out as written: `Cancel` builds `hubCancelDeps` over the real hub, so no tier0 row of `TestCancelRun` reaches it, and a `TestRemove` row over a fake `Cancel` only repeats that row's existing "another cancel error" case.
  The new helper puts the end-only conversion where `TestCancelRun`'s fakes reach it.
- F2 (MEDIUM), fixed as proposed: card 11's `reconcileWarnings` line names `run lym fabric pull in <worktree>`, then the re-run of the same `lym adopt`, with the reconcile's own `WayForward` following as detail in parentheses.
  The `TestAdopt` row for a diverged adopted-beam branch now asserts that the warning names `lym fabric pull`, with that way forward following it.
  The self-check challenges this proposal; see Deferred.
- F4 (LOW), fixed as proposed and widened by the self-check.
  - Card 11's Edit list adds `heldBeamWayForward`, `ownerWriteError`, `runBeamSteps`, `beamcli#createCmd`, `battencli#OwnedElsewhereError.Error` and `internal/beamcli/doc.go`.
  - Card 11's Intent makes each of these, together with `adopt`'s lost-owner and read-back errors and `adoptcli/doc.go`'s nested-beam bound and landing-hold paragraph, name the subject's adopt check as what resumes the runs.
  - Card 12's Edit list adds `landingshed#ownerHaltReason` and `internal/battenshed/doc.go`, and its Uses list adds `internal/landingshed/doc.go`.
    Its Intent gains lines for the Finalize owner refusal and for battenshed's detached-first-pass line.
- F5 (LOW), fixed as proposed: card 5's Edit list adds `internal/gateslot/doc.go`, and its Docs paragraph says the gate table's `adopt` row names `lym adopt check <subject>`, run in the subject's worktree.
- F6 (LOW), fixed as proposed: card 5's Intent rewrites `check.go`'s header to name the gate run in the subject's worktree.
  Card 11's Intent rewrites `adopt.go`'s header to name the hold before the owner write, with no lift or resume, and `deps.go`'s header to drop the run and child resume seams.
  No file self glyph was added.
- F7 (LOW), fixed through the proposal's second option: card 6 gains a Bound line.
  The `OwnedHere` false branch runs only in the ticks between an owner move and dobby's next probe read of the owner.
  A record a machine keeps after losing the subject can run one redundant `adopt-check` if that machine adopts the subject again.
- F8 (LOW), fixed as proposed: card 13's case 1 treats as open an own fix task whose entry holds a run status with no readable batten run record, and names it as starting.
  `TestFailedEnd` gains that row.
- F9 (LOW), fixed as proposed: card 11's `TestAdopt` row adopts with `--keep-local` over a live pair under a beam owned elsewhere, and asserts that the run gets no hold and reports running here.
- F10 (LOW), fixed as proposed: card 5 re-signs `tierNotice` as `func tierNotice(subject string, tier CheckTier, logPath string) string` and `suiteMessage` as `func suiteMessage(checks []suiteStart) string`.
  The Intent says `checkMarkOf` reads the subject from `CheckRecord.Subject` and keeps its signature.
- F11 (NIT), fixed as proposed: card 3's Intent says `FileFrontTask` refuses a payload carrying `front` or `on_land` with `checkFrontPayload`'s payload refusal text before the upsert, then upserts with `checkFrontPayload` skipped.

Card 13 also carries the parent's recorded decision "The fix task's gate label only where the board knows it":
- labels `bug` and `gate` only when the hub's board vocabulary knows `gate`, read through a `Known` seam over `boardengine.LoadConfig` and `Vocabulary.Known`, else `bug` alone;
- the matching `TestFailedEnd` row;
- the two Uses glyphs.

After all changes, `lym loom validate-plan` reads ok.

## Self-check

- fix-proposal, MEDIUM: F2's proposal makes `lym fabric pull` the first step even for an `ahead` branch, where pull has nothing to move, and for a diverged code branch, which pull does not settle; only the reconcile's own way forward (`git merge` and push) settles it.
  Disposition: deferred, see below.
- fix-consistency, LOW: card 11's ImpactSummary said "all inside adoptcli". Fixed as proposed: it now names beamcli's doc and create help and battencli's owned-elsewhere refusal.
- fix-consistency, LOW: card 12's ImpactSummary missed landingshed and battenshed. Fixed as proposed.
- fix-consistency, LOW: card 13's ImpactSummary missed the board vocabulary read. Fixed as proposed.
- fix-consistency, LOW (noted for fix-claims): card 6's new Bound put the window at a single owner read, whereas `dobbyengine/doc.go:39` has dobby read the owner at start and on each probe tick.
  Fixed as proposed: the Bound names the ticks between an owner move and the next probe read.
- fix-claims, LOW: card 4's `TestCancelRun` line asserted `--remove` on every end-only row, but those rows run with `remove` both false and true (`cancel_test.go:784-791`).
  Fixed as proposed: the rows assert `cancelCommand(slug, remove)`, with `--remove` only on the remove rows.
  The Intent's error text names `<command>` as `cancelCommand(slug, remove)`.
- fix-affected-set, LOW: `beamcli#createCmd`'s long help (`beamcli/cli.go:499`) repeats the stale `lym adopt` sentence. Fixed as proposed on card 11.
- fix-affected-set, LOW: `battencli#OwnedElsewhereError.Error` (`battencli/owner.go:102`) says adopt resumes the run. Fixed as proposed on card 11.
- fix-affected-set, LOW: `runBeamSteps`' doc comment (`adopt.go:627`) says the parent's adopt resumes the landing. Fixed as proposed on card 11.
- fix-affected-set, LOW: `adoptcli/doc.go:63,66` say adopt resumes. Fixed as proposed in card 11's Intent.

Every fork returned a report.

## Disputed

None.

## Deferred

- Self-check fix-proposal finding on F2: the premise holds.
  `BranchAhead` means origin's head is an ancestor of the local tip (`internal/fabricengine/originreconcile.go:130`), so pull moves nothing.
  A diverged code branch's own way forward is `git merge origin/<branch>` then a push (`internal/fabricengine/failedpush.go:258-266`).
  The sound change leads with the reconcile's `WayForward` for `diverged` and `ahead` and keeps `lym fabric pull` for `rewritten`.
  That departs from D2's wording, "its branch, its state and `lym fabric pull` in that worktree", and from its acceptance criterion, "a warning naming `lym fabric pull`".
  Deferred for a parent decision on whether D2's way forward may follow the branch state.
  Card 11 keeps D2's literal wording until that decision is made.
