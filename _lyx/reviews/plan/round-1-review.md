---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: handler
    location: _lyx/plan/05-reconcile-owner-outcome.md:41
    summary: The owner guard on retryWeftPush (cards 5 and 6) reads origin's Bolt outside ReconcileOrigin. This breaks the decision record's Constraint that every other owner read uses the local Bolt, and no Decision covers it.
    fix: |
      Card 5: drop these three targets:
        - Edit `internal/fabricengine#retryWeftPush`
        - Create `plan:internal/fabricengine#InstallOwnerReader`
        - Create `plan:internal/fabricengine_test#TestPushAnchored_OwnedElsewhereWeftIsNotReplayed`
      Also drop the Intent sentences that start "The same replay recurs on the old machine's own push" and "`Fabric.Pull` is left as it is", the TestPushAnchored sentence, and "and the guarded push retry" from the doc.go sentence.
      Card 6: drop Edit `cmd/lyx#newRoot`, drop Uses `plan:internal/fabricengine#InstallOwnerReader`, and drop the Intent sentence that starts "`cmd/lyx`'s `newRoot` pre-run installs".
      00-overview.md: delete the Shared Decisions bullet at line 41. Keep its continuation line 42, the `Pattern-Departure:` trailer sentence, by moving it under the PATTERN-owner-machine bullet above it.
      If the fixer judges that the weft-replay hole must close in this task, it instead asks the parent (ly:orch) for an Added-after-Discussion entry in the decision record that licenses the guarded push retry and its origin-Bolt read, and keeps the cards only once that entry exists.
      The combined validate-plan overlay of these target drops reported no new blocking finding.
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/16-adopt-module.md:52
    summary: Card 16 adds internal/adoptcli, a module cli with no engine of its own. PATTERN-cli-cobra's background lists every such deviation from the cli-imports-its-engine rule, and no card adds adoptcli to that list.
    fix: |
      Card 16, Edit group: add `pattern/PATTERN-cli-cobra.md` after `docs/overview.md`.
      Card 16, Intent: add a sentence saying that `pattern/PATTERN-cli-cobra.md`'s Package naming list gains a deviation bullet beside `cleanupcli`'s:
        `adoptcli` calls the reconcile, owner setter, fabric add, tender lift and batten resume functions in-process, with no engine package of its own.
      This addition passed the validate-plan overlay.
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/03-board-owner-writes.md:51
    summary: SetOwners retries a deferred push after a fixed one second, and TestSetOwners has a row in which the push lock is held across that retry. The test therefore waits out a production duration, which PATTERN-test-speed forbids (injected clocks).
    fix: |
      Card 3, Intent: make the retry wait injectable. For example, give `OwnerRequest` or a package-level seam a `RetryDelay` that defaults to one second.
      State that TestSetOwners' push-lock row sets it to zero, so no row waits a real second.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/10-innerrun-owner-check.md:40
    summary: ownerDue, the sleep watch and the previous coverage live on childWait, which every Call builds fresh. A sleep detected on the check where the waking child also changed state ends the wait before ownerStep runs, and the next Call's new watch never sees the sleep, so D6's post-wake owner check is lost.
    fix: |
      Card 10, Intent:
        - Keep the sleep watch, `ownerDue` and the last pause coverage on `innerRunProducer`, as `ownedBy` already is, so they survive across Calls.
        - Run the owner step before `checkChild` can end the wait.
      Card 10, TestInnerRun_OwnerCheck: add a row in which a sleep and a child state change arrive on the same check, and the next Call still runs the owner check.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/08-batten-owner-check.md:50
    summary: Card 8's resume claims the prime when none is recorded, through SetOwners, which needs origin. The card states no disposition for a failed claim (origin unreachable, a refused push, or a diverged or dirty pull), and it does not order the claim against the unreachable branch. On a hub an older binary left with no prime owner, D3's Warn-and-resume for an unreachable origin is then unspecified.
    fix: |
      Card 8, Resume paragraph:
        - The prime claim runs only on a reachable reconcile.
        - With origin unreachable the resume makes no owner write and keeps D3's Warn and resume.
        - A failed claim on a reachable origin (unreachable push, refused push, or diverged or dirty pull) warns and goes on to the owner verdict, as card 13's orch-start claim does.
      Card 8, TestRunCmd_ResumeDispositions: add a row in which an unreachable reconcile on a hub with no prime owner warns and resumes with no write.
      Card 8, TestRunCmd_ResumeDispositions: add a row in which a failed prime claim on a reachable reconcile warns and still reaches the owner verdict.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/plan/17-orch-role-and-pattern.md:15
    summary: Card 17's short PATTERN.md line drops the declared entry's exception for a Finalize past its parent-side merge, and implies that every owner judgment happens in ReconcileOrigin. The overview line agents get inlined is then stricter than the declared text, and future reviews would read the carve-out in cards 10 to 12, and the local-Bolt reads in cards 8 and 12, as violations.
    fix: |
      Card 17: replace the fenced entry line with the line below, which is 353 characters (under pattern.MaxEntryLineChars and no longer than the card's current line):
      - `PATTERN-owner-machine` — Starting, resuming or landing a run, bar a Finalize past its parent-side merge, or acting after a wake: only where its entry reads owned by no other machine; owner set only by `boardengine`'s setter; origin verdicts in `ReconcileOrigin`; name only from `internal/machinename`. — [background](pattern/PATTERN-owner-machine.md)
  - id: F7
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/01-machine-name.md:3
    summary: Card 1 adds the stdlib-only leaf internal/machinename, which the decision record binds under PATTERN-leaf-packages. It adds no section for it to pattern/PATTERN-leaf-packages.md, where every leaf's closed import set is listed, and no entry to docs/overview.md's module tree, which lists the other leaf packages.
    fix: |
      Card 1: add an Edit group with `pattern/PATTERN-leaf-packages.md` and `docs/overview.md`, and an ImpactSummary line: two docs gain the new leaf package; no code outside the new package changes.
      Card 1, Intent: say that the leaf background gains an `internal/machinename` section ("Imports only the standard library."), and that the overview's module tree gains `internal/machinename/`.
      This addition passed the validate-plan overlay.
  - id: F8
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/09-loom-owner-check.md:32
    summary: ReconcileOrigin classifies only pairs with a batten run record (recordedPairs). A loom or darn run with no batten record therefore never reads owned_elsewhere at loom start, resume or Finalize, and neither card 9 nor card 12 states that bypass. TestRunOwnerVerdict's owned-elsewhere row is reachable only if it seeds a batten status file, which the card does not say.
    fix: |
      Cards 9 and 12, Intent: add a Bound sentence saying that a run whose pair has no batten run record reads owned here, since the reconcile judges only recorded pairs.
      Card 9: say that TestRunOwnerVerdict seeds a batten status file at the prime for the pair, so its owned-elsewhere rows reach the verdict.
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/03-board-owner-writes.md:22
    summary: Several public fabricengine and tendercli symbols that other packages reuse are bundled into consumer cards instead of getting cards of their own, namely ParentSlug (card 3, used by six later cards), LocalOnlyCommits (card 12), DropLocal and its result (card 16, a new destruction primitive) and PauseOwnedElsewhere (card 11).
    fix: |
      Move each symbol, with its test, into a card of its own, inserted before its first consumer with `lyx loom insert-card`:
        - `fabricengine.ParentSlug` and `TestParentSlug`, before card 3.
        - `fabricengine.Fabric.LocalOnlyCommits` and `TestLocalOnlyCommits`, before card 12.
        - `fabricengine.DropLocal`, `DropLocalResult`, the destroy.go header and the destructive-guard table rows, with `TestDropLocal`, before card 16.
        - `tendercli.PauseOwnedElsewhere`, before card 11.
      Then remove those targets from the consumer cards and leave them under Uses.
  - id: F10
    severity: LOW
    class: decision
    origin: lens:way-forward
    location: _lyx/plan/16-adopt-module.md:60
    summary: Step 2 rewords fabric's leftover-branch refusal to delete the leftover local branch and re-run adopt. D5 step 2 says the slug stops with fabric's own message and way forward, and the leftover branch can hold this machine's unpushed commits, which the reworded step drops with no archive, unlike --drop-local.
    fix: |
      Card 16, Step 2: keep fabric's own refusal message and way forward, as D5 step 2 says.
      If a reword is kept instead, it names archiving the leftover tip under `archive/<slug>/<tip>` before the branch is deleted.
  - id: F11
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/11-batten-innerrun-wiring.md:34
    summary: The TestChildSpawnError row for an owned_elsewhere envelope with no owner field tests a state no writer produces. Card 9's refusal always writes owner, from a verdict whose Elsewhere requires a non-empty owner.
    fix: |
      Card 11, Intent: drop the no-owner row and the "another machine" wording, or state that the empty-owner fallback is a fail-loud wiring guard that takes no reaching test.
---

### [BLOCKING] The push-retry owner guard is outside the answer key and reads origin's Bolt

**Location:** `_lyx/plan/05-reconcile-owner-outcome.md:41`, `_lyx/plan/06-owner-seam-wiring.md:27`, `_lyx/plan/00-overview.md:41`

**Issue:** D4 puts the fast-forward-only rule inside the origin reconcile.
The Constraints section says the owner verdict that gates a resume, a wake or a landing is computed inside `ReconcileOrigin`, and that "every other owner read reads the local Bolt", listing those reads.
Card 5 adds a new owner gate on `retryWeftPush` (`pushclassified.go:463`), which reads `board.json` from origin's freshly fetched Bolt through a process-installed reader that card 6 installs in the root pre-run.
No Decision carries a push-path owner guard.
The decision record's Open risks say the old machine's pushes "are refused and recorded as failed pushes", but today's retry replays a weft onto origin's tip instead, so the plan closes a gap the record's premise missed by adding unlicensed scope with a read source the Constraint excludes.
That is a design change, and it belongs with the parent.

### [BLOCKING] adoptcli is an unlisted PATTERN-cli-cobra package-naming deviation

**Location:** `_lyx/plan/16-adopt-module.md:52`

**Issue:** `pattern/PATTERN-cli-cobra.md`'s Package naming section says that a `<module>cli` imports its `<module>engine`, and lists every deviation, `cleanupcli`'s "with no engine package of its own" among them.
Card 16 creates `internal/adoptcli` with "no engine of its own", and no card adds it to that list, so the change breaks the entry as written.

### [BLOCKING] SetOwners' one-second retry is waited out by a test row

**Location:** `_lyx/plan/03-board-owner-writes.md:51`

**Issue:** Card 3 fixes the push retry at "once after one second", and its TestSetOwners has a row "a push lock held across the retry refusing naming `lyx board sync`".
Nothing makes the delay injectable, so that row waits a real production second.
`PATTERN-test-speed` requires injected clocks and says that no test waits out a production duration.

### [MEDIUM] InnerRun's owner trigger dies at a Call boundary

**Location:** `_lyx/plan/10-innerrun-owner-check.md:40`

**Issue:** `childWait` is built fresh by each `Call` (`innerrun.go:466`), and its `sleep` watch is fed in `notice.go:65`, after `checkChild` (`innerrun.go:698-746`).
`checkChild` ends the wait on any change in the child's state.
A machine that wakes from a suspend can see the child's state change on the same check that records the sleep.
In that case the wait returns before `ownerStep` runs, and the next Call's new `SleepWatch` starts unseen and records no sleep.
The same loss hits an unreachable verdict's "keeps `ownerDue` for the next check" at any Call boundary, and a pause lifted between Calls.
This post-wake check is the core of D6.

### [MEDIUM] Card 8 leaves the resume's failed prime claim undecided

**Location:** `_lyx/plan/08-batten-owner-check.md:50`

**Issue:** `reconcileResume` "claims the prime when none is recorded", and the card gives no disposition or way forward when that write fails.
It also does not say whether the claim runs when the reconcile reports origin unreachable.
Every hub upgraded to this binary starts with no prime owner, so the first resume with origin unreachable either refuses, against D3's "a Warn and the resume", or skips the claim, which the card does not state.
The card's test rows cover only a successful claim.
Card 13's orch-start claim, by contrast, warns on failure.

### [MEDIUM] The short PATTERN.md line drops the declared exception

**Location:** `_lyx/plan/17-orch-role-and-pattern.md:15`

**Issue:** The declared text bars "the rest of a Finalize already past its parent-side merge" from the rule.
It also names where each non-reconcile owner read comes from: the pulled Bolt, the pushed Bolt, the Bolt the reconcile left, and the local Bolt.
Card 17's overview line keeps neither.
It states the rule with no exception, and with "judged against origin in `ReconcileOrigin`" as the only source.
The card's own cut rule for the other lines removes "no clause ... exception", but its new line does.
Once the declared list stops overriding the overview, which happens at landing, the Finalize carve-out in cards 10 to 12 reads as a breach of PATTERN.

### [MEDIUM] internal/machinename is missing from the leaf-package background and the module tree

**Location:** `_lyx/plan/01-machine-name.md:3`

**Issue:** The decision record's Constraints bind the new package under `PATTERN-leaf-packages`.
That entry's background gives every leaf package a section with its closed import set, `internal/pauserecord` and `internal/lyxtemp` among them, and `docs/overview.md`'s module tree lists those leaves too.
Card 1 creates the package and its allowlist test but adds neither section nor tree entry.

### [LOW] Runs with no batten record bypass the owner verdict unstated

**Location:** `_lyx/plan/09-loom-owner-check.md:32`, `_lyx/plan/12-finalize-owner-checks.md:43`

**Issue:** `classifyEntries` judges only `recordedPairs`, which are pairs with a batten status file at the prime (`originreconcilemoves.go:435-455`).
Card 9 reads "the run's owned_elsewhere entry", and card 12 takes "the owned_elsewhere entry's owner".
A run with no batten record therefore reads owned here at both, and neither card's Intent states that bound.
TestRunOwnerVerdict, an integration test over a hubforge hub, reaches its owned-elsewhere rows only if it seeds a batten status file.

### [LOW] Public cross-package symbols are bundled into consumer cards

**Location:** `_lyx/plan/03-board-owner-writes.md:22`, `_lyx/plan/12-finalize-owner-checks.md:23`, `_lyx/plan/16-adopt-module.md:18`, `_lyx/plan/11-batten-innerrun-wiring.md:12`

**Issue:** The plan spec's granularity rule gives a symbol that is part of a module's public surface, or reused across packages, its own card.
`fabricengine.ParentSlug` is consumed by cards 4, 6, 8, 9, 12, 14 and 16 but sits in the board-writes card.
`Fabric.LocalOnlyCommits` sits in the Finalize card, and `DropLocal`, a new destruction primitive with its own guard-table rows and test, sits in the adopt card.
`tendercli.PauseOwnedElsewhere`, which battencli and the root pre-run use, sits in the batten wiring card.

### [LOW] Step 2's reworded leftover-branch refusal departs from D5

**Location:** `_lyx/plan/16-adopt-module.md:60`

**Issue:** D5 step 2 stops the slug "with fabric's message and way forward".
The card replaces that wording with batten's style of reword, which names deleting the leftover local branch.
For adopt, that branch can be this machine's own earlier work: `lyx fabric remove` keeps a code branch.
Unlike `--drop-local`, which tags each local tip before moving it, the reworded step loses those commits.

### [LOW] An unreachable TestChildSpawnError row

**Location:** `_lyx/plan/11-batten-innerrun-wiring.md:34`

**Issue:** The row for an `owned_elsewhere` envelope with no `owner` field guards a shape no writer produces.
Card 9's `refuseOwnedElsewhere` always writes `owner`, from a verdict whose `Elsewhere` is `owner != "" && owner != here`.
The kind is new, so no older binary writes it either.

## Rejected

- Pinning `adoptcli.RunCLI` and `RunCLIIn` in `cmd/lyx/seamsignature_test.go` (affected-set): no PATTERN-cli-cobra clause requires that pin, since "Per-CLI copies are no longer an obligation". The file's blank-identifier slices name no member a card could target, so `edit-test-file` leaves no admissible fix.

## Focus notes

- Card order across cards 2 to 6 and 8 to 13: every owner reader, `OwnerRequest`, `InstallOwnerPause` and owner-pause-label use comes from an earlier card. Not a defect.
- Attack surface:
  - `OwnerRequest.Takeover` is bounded in card 3 ("only `lyx adopt` moves an owner") and in the Shared Decisions.
  - The Finalize carve-out is bounded by the `ParentHoldsTaskWork` condition in card 12 and by card 10's Bound line.
  - The nil-seam-is-absent rules are bounded to hand-built test receivers, with production wiring pinned by `TestLandingDeps_EveryFieldPopulated`, `newLoomCLI` and `Command`.
  - None of these is a defect.
- Member glyphs and re-sign arrows on cards 3, 5, 6, 10, 12 and 16: the one re-signed member, `classifyEntries`, carries its arrow. Every other Intent-named member keeps its signature. Not a defect.
- Test economy on cards 7, 10, 14 and 16: the new tests cover behavior no existing test covers. Card 7 extends existing tests. Not a defect.
- Card 17's `Pattern-Departure:` trailer documents the short form the Added-after-Discussion entry decided, so it is accepted. The finding against card 17 is about the line's content.
