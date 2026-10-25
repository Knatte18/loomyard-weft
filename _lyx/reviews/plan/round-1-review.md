---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: >-
      _lyx/plan/09-weft-pull-and-push-stop-on-marker-change.md, _lyx/plan/13-move-record-stops-weft-writers.md (internal/fabricengine/parenttips.go:270)
    summary: >-
      The second weft pull, Fabric.fastForwardWeft, reached from FastForwardPair in landingshed's Finalize, runs PullWeft with neither the D10 marker comparison nor the D7 move-record stop, so a landing can change a parent's checked-out marker or move its weft mid-move.
    fix: >-
      Add `internal/fabricengine#Fabric.fastForwardWeft` and `internal/fabricengine_test#TestFastForwardPair_MovesBehindPairToUpstream` to the Edit lists of card 9 and card 13. In card 9's Intent, fastForwardWeft refuses an empty own marker, fetches, and runs `weftAnchorStop` on HEAD and the fetched upstream before `PullWeft`. On a stop it skips the weft pull with its existing warning, naming the way forward, and leaves the code side to go on as it does today. In card 13's Intent, it skips the weft pull the same way while `MoveInProgress` reports a standing record. Each card adds one row to `TestFastForwardPair_MovesBehindPairToUpstream`: an upstream carrying another marker, and a standing record, each leave the weft HEAD unchanged. Checked with `lyx loom validate-plan --overlay`, with no new finding.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: >-
      _lyx/plan/17-fabric-anchor-verb.md (owner and unfinished-run refusals), _lyx/plan/11-origin-reconcile-stops-on-marker-change.md, _lyx/plan/10-checkout-and-drop-local-refuse-another-anchor.md
    summary: >-
      For a worktree the board says another machine owns, the owner refusal names lyx adopt and the unfinished-run refusal names lyx batten cancel. Cancel names adopt for an owned-elsewhere slug, and adopt stops on the anchor_stopped weft or the drop-local marker mismatch, naming the refused --set again. The always-open exit, --set to the fetched upstream tip's marker, is never named.
    fix: >-
      In card 17, the owner refusal, and the unfinished-run refusal when the board names another owner, also name `lyx fabric anchor <slug> --set <marker at the fetched upstream tip>`, which the owner refusal lets through. TestChangeAnchor's owner-refusal row asserts that text.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: >-
      _lyx/plan/15-anchor-change-engine.md (Apply failure), _lyx/plan/17-fabric-anchor-verb.md (Apply error path)
    summary: >-
      An Apply failure after the move commit (wiring at B, launchers, workspace files) has no prescribed end state. Card 15 restores only failures before the commit, and card 17 restarts the ended dobbies, which card 16's ensureOne gate refuses while the record still stands. The error names no landed re-run.
    fix: >-
      Card 15 states that an Apply failure after the commit, or after the integration moved the branch, leaves the move record standing and returns an error naming the landed re-run `lyx fabric anchor <slug> --set <B>`. Card 17's Apply-error path restarts dobbies only for a failure before the commit. After the commit it releases the parent lock and returns the error, leaving the dobby start to the re-run's Finish. Both tests gain a row: a hook failure after the commit leaves the record and is finished by a re-run.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:way-forward
    location: >-
      _lyx/plan/15-anchor-change-engine.md (validateAnchorTarget), _lyx/plan/18-beam-create-at-an-anchor.md (--anchor validation)
    summary: >-
      The B validation refusals name no way forward for an invalid value, a directory tracked neither in the warp nor upstream, a link or reserved segment, or a real wired name under B. The decision record's Constraints require every refusal to name its way forward.
    fix: >-
      In card 15's validateAnchorTarget, each refusal names its next step. An invalid or reserved-segment value names choosing another directory. A directory not tracked names committing it in the warp checkout first. A real wired name under B names moving it out of B in the warp. Card 18 says its --anchor refusals carry the same text, and TestAnchorChange's refusal table asserts each.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: >-
      _lyx/plan/02-location-literal-scan.md (driverLockBusyError fallback), _lyx/plan/04-consumer-target-locations.md (Teardown.taskAnchor fallback and its invalid-marker row)
    summary: >-
      The fallbacks for an invalid own marker in driverLockBusyError and Teardown.taskAnchor are unreachable. On that input, taskLocation's ResolveWorktree, in probeQuiet and in driverLockHeldIn, errors first (pairteardown/teardown.go:580, :662). Card 4's invalid-marker teardown row cannot pass, and card 2's claim that card 4 reaches the fallback is false.
    fix: >-
      Drop the NamedWorktree-error fallbacks from card 2's driverLockBusyError and card 4's Teardown.taskAnchor, building both texts from the pair Location the teardown already resolved. Drop card 4's second TestRun_TeardownScenario step (the invalid-marker pair) and card 2's sentence that card 4 reaches the fallback.
  - id: F6
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: >-
      _lyx/plan/06-clone-records-marker-on-prime-weft.md, _lyx/plan/15-anchor-change-engine.md (cmd/lyx/uncontainedwrite_test.go:70)
    summary: >-
      The new raw marker writes in fabricengine fall under the tier0 per-file write allowlist in cmd/lyx/uncontainedwrite_test.go. These are commitAnchorMarker's write into a weft root and the D7 move's write of marker B. No card names that allowlist, and the clone.go entry's reason, that the marker is written into the _board worktree, becomes false.
    fix: >-
      Add `cmd/lyx#TestNoUncontainedWrite_FabricengineProductionSource` to card 6's Edit list and to card 15's Edit list. Card 6 names the file commitAnchorMarker lives in and rewords the clone.go entry's reason to the prime weft root. Card 15 adds an entry for the file holding the move's marker write, with its reason. Checked with `lyx loom validate-plan --overlay`, with no new finding.
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: >-
      _lyx/plan/10-checkout-and-drop-local-refuse-another-anchor.md (DropLocal)
    summary: >-
      Card 10 compares drop-local's anchors "through weftAnchorStop's rule". That rule never stops when the local tip already contains origin's tip. Yet D10 refuses the drop-local reset whenever origin's tip carries a marker other than the checked-out one, as when a local re-anchor is not yet pushed and the reset would change the checked-out marker.
    fix: >-
      In card 10, DropLocal refuses whenever treeAnchor of origin's tracking tip differs from the checked-out anchor, whatever the ancestry. Give weftAnchorStop, in card 9, a mode or a sibling result for a reset, so the one comparison function still serves drop-local as D10 says. Add a TestDropLocal row: the local weft ahead of origin with an unpushed re-anchor refuses, with no tag and both branches unmoved.
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: >-
      _lyx/plan/18-beam-create-at-an-anchor.md (Topology.add with Anchor)
    summary: >-
      Card 18 lists BeginAnchorChange, Apply and Finish under Uses, yet its Intent re-describes the step-4 move inside Topology.add and wires the pair itself. It leaves open which engine call performs the move. If it is BeginAnchorChange, that refuses a fork point without a committed marker, as on a hub not yet migrated, after the pair is forked, and Apply's wiring, launchers and workspace writes would duplicate Add's own.
    fix: >-
      Card 18's Intent names the one engine path the create at an anchor takes, for example a step-4 move helper shared with AnchorChange.Apply, run by Add after the fork and before any wiring. It states that the committed-marker and dirty-weft refusals of BeginAnchorChange do not apply to the freshly forked pair, or that a markerless fork point is refused before the fork naming `lyx fabric reconcile`. TestAdd_AtAnchor gains the matching row.
  - id: F9
    severity: MEDIUM
    class: design
    origin: handler
    location: >-
      _lyx/plan/15-anchor-change-engine.md (not-landed recovery and failure restore)
    summary: >-
      The recovery deletes what HEAD does not track under B/<names>, and the failure path restores the working tree. Neither is routed through the destruction chokepoint in destroy.go, which PATTERN-fabric-git reserves for every destructive primitive in fabricengine. Card 15 names only the junction removal as going through it.
    fix: >-
      Card 15 routes the deletion of untracked B/<names> paths through a new destroy.go executor, adding `plan:internal/fabricengine#removeUntrackedMovePaths` -> `func removeUntrackedMovePaths(rec *Mutations, weftPath string, paths []string) error` under Create. Its gates are containment under the weft worktree, ownership of the move's pathspecs, and dirtiness not applicable, since the record proves the paths are the move's. The Intent says the restore goes through it too. Checked with `lyx loom validate-plan --overlay`, with no new finding.
  - id: F10
    severity: MEDIUM
    class: decision
    origin: handler
    location: >-
      _lyx/plan/17-fabric-anchor-verb.md (owner refusal exemption), _lyx/plan/00-overview.md (Shared Decisions, Every way forward progresses)
    summary: >-
      Card 17 exempts a worktree with a standing move record from D7's owner refusal. D7 lists that refusal without the exemption, and the decision record carries no Added after Discussion entry for it. The exemption has a sound reason, since adopt refuses while the record stands, but the answer key does not license it.
    fix: >-
      Through the parent, have the decision record gain an Added after Discussion entry recording the exemption, bounded to the record's own B, and keep card 17 as written. If the parent declines, drop the exemption from card 17 and the Shared Decisions bullet, and name the way forward the parent records instead.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: >-
      _lyx/plan/18-beam-create-at-an-anchor.md (record removal)
    summary: >-
      Card 18 has Topology.add remove the create-at-anchor record "once the remaining create steps are done", but createSteps' push, workspace regeneration and dobby start run after AddPair returns. A record still standing then makes card 16's ensureOne refuse the create's own dobby start.
    fix: >-
      Card 18 states that Topology.add removes the move record before it returns, after the wiring, scratch link, launchers and origin record, and so ahead of createSteps' push, workspace and dobby steps. A re-run after a kill past that point passes EnsureAnchor's equal-anchor branch.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: >-
      _lyx/plan/06-clone-records-marker-on-prime-weft.md (internal/fabricengine/unwire.go:5, internal/fabriccli/fabric.go:423, internal/fabricengine/codebinding.go:2, internal/fabricengine/clone.go:33)
    summary: >-
      The doc comments and unwire help that call .lyx-anchor a repo-wide weft:main record beside .lyx-warp stay unchanged, though card 6 moves the marker to each weft root.
    fix: >-
      Add to card 6's Edit list `internal/fabricengine/unwire.go` and `internal/fabricengine/codebinding.go` (file headers), `internal/fabriccli#Command` (the unwire Long help at fabric.go:423, inside Command) and `internal/fabricengine#staleFabricAnchorName` (its doc). Card 6's Docs sentence says each now places the marker at each worktree's weft root. Checked with `lyx loom validate-plan --overlay`, with no new finding.
  - id: F13
    severity: LOW
    class: decision
    origin: handler
    location: >-
      _lyx/plan/13-move-record-stops-weft-writers.md (Topology.Checkout)
    summary: >-
      Card 13 adds a checkout refusal while a move record stands. D7 step 1 lists the writers a standing record stops, and checkout is not among them, and no Added after Discussion entry licenses it. The guard is protective, since recovery acts on the branch the record was written on.
    fix: >-
      Through the parent, have the decision record gain an Added after Discussion entry adding checkout to D7 step 1's list, and keep card 13's checkout refusal. If the parent declines, drop the checkout refusal, its Edit glyphs and its TestCheckout_RefusesAnotherAnchor row from card 13.
---

### [BLOCKING] A landing's weft fast-forward bypasses both anchor stops

**Location:** cards 9 and 13; `internal/fabricengine/parenttips.go:270` (`Fabric.fastForwardWeft`), reached from `FastForwardPair` at `internal/landingshed/finalize.go:400`.

**Issue:** Cards 9 and 13 guard only `Fabric.pullWeftArm`.
`fastForwardWeft` calls `f.PullWeft(opts)` directly, so a Finalize that fast-forwards a parent pair can pull an upstream carrying another marker, changing the checked-out marker without a rewire, which D10 forbids for every weft pull.
It also pulls while a move record stands, before the anchor verb takes the parent lock, which D7 step 1 forbids.

### [MEDIUM] The owned-elsewhere way forward loops

**Location:** card 17's owner and unfinished-run refusals; cards 10 and 11.

**Issue:** For a worktree another machine owns, adopt stops on the `anchor_stopped` weft (card 11) or the drop-local mismatch (card 10) and names the `--set`.
The `--set` is refused by the owner refusal, which names adopt, and the unfinished-run refusal names `lyx batten cancel`, which names adopt for an owned-elsewhere slug (`internal/battencli/cancel.go:389`).
The one step that progresses is `--set` to the fetched upstream's marker, which D7 leaves open to every machine, and no refusal names it.

### [MEDIUM] An Apply failure after the move commit has no end state

**Location:** card 15's failure restore; card 17's `Apply`-error path; card 16's `ensureOne` gate.

**Issue:** Card 15 restores only failures before the commit.
After the commit the record stands, so card 17's restart of the ended dobbies is refused by card 16's gate, and the error names no landed re-run.

### [MEDIUM] B validation refusals name no way forward

**Location:** card 15's `validateAnchorTarget`; card 18's `--anchor` validation.

**Issue:** Only the case where the warp's upstream tracks `B` names a next step.
Every other validation refusal names none, against the Constraint that every refusal names its way forward.

### [MEDIUM] Unreachable invalid-marker fallbacks in teardown

**Location:** card 2 (`driverLockBusyError`); card 4 (`Teardown.taskAnchor`, second `TestRun_TeardownScenario` step).

**Issue:** With an invalid own marker, `taskLocation`'s `ResolveWorktree` fails in `probeQuiet` and in `driverLockHeldIn` (`internal/pairteardown/teardown.go:580`, `:662`) before either busy text is built.
The fallback is never reached, and card 4's row asserting it cannot pass.

### [MEDIUM] The fabricengine write allowlist is outside every card

**Location:** cards 6 and 15; `cmd/lyx/uncontainedwrite_test.go:70`.

**Issue:** `TestNoUncontainedWrite_FabricengineProductionSource` fails on a raw `os.WriteFile` in a fabricengine file without an allowlist entry.
`commitAnchorMarker` and the move's marker write are new raw writes, and the `clone.go` entry's reason names the `_board` worktree, which card 6 makes false.

### [MEDIUM] Drop-local's comparison misses a local re-anchor ahead of origin

**Location:** card 10 (`DropLocal`).

**Issue:** D10: drop-local "refuses before tagging or moving any branch when origin's tip carries a marker other than the checked-out one".
`weftAnchorStop` stops only when the upstream has commits the head lacks.
A drop-local over a weft holding an unpushed re-anchor resets to origin's older marker without a refusal, if card 10 follows "weftAnchorStop's rule" as written.

### [MEDIUM] Card 18 leaves the create-at-anchor engine path open

**Location:** card 18 (`Topology.add` with `Anchor`).

**Issue:** The card uses `BeginAnchorChange`, `Apply` and `Finish`, but its Intent describes Add moving content, committing and wiring itself.
`BeginAnchorChange` refuses a weft with no committed marker, which a fork from an unmigrated hub's weft is, after the fork has already happened.
`Apply` also wires, writes launchers and regenerates workspace files, which Add then repeats.
D6's rationale, that one move step serves create and change, needs the card to name the shared step.

### [MEDIUM] The recovery's deletions bypass the destruction chokepoint

**Location:** card 15 (not-landed re-run, failure restore).

**Issue:** `PATTERN-fabric-git`: destroying in fabricengine happens only in `destroy.go`.
Card 15 deletes untracked paths under `B/<names>` and restores the working tree, without naming a `destroy.go` executor for either.

### [MEDIUM] The owner-refusal exemption is not licensed by the decision record

**Location:** card 17; overview Shared Decisions.

**Issue:** D7's owner refusal has no exemption for a standing move record.
The plan's exemption avoids a real deadlock, since adopt refuses while the record stands, but it changes a Decision with no `Added after Discussion` entry.

### [LOW] Card 18 does not time the create record's removal

**Location:** card 18.

**Issue:** "Once the remaining create steps are done" can't be done inside `Topology.add`, because those steps run in `createSteps` after `AddPair` returns.
A record still standing makes card 16's gate refuse the create's dobby start.

### [LOW] Stale docs keep the marker on weft:main

**Location:** card 6; `unwire.go:5`, `fabric.go:423`, `codebinding.go:2`, `clone.go:33`.

**Issue:** These comments and the unwire help still describe `.lyx-anchor` as a repo-wide `weft:main` record.

### [LOW] Checkout's standing-record refusal is not licensed

**Location:** card 13 (`Topology.Checkout`).

**Issue:** D7 step 1 lists the writers a standing record stops, and checkout is not among them.
The guard is sound, but it adds scope the answer key does not license.

## Focus verdicts

- Card order across cards 13–19: no card uses a symbol a later card creates. `ErrAnchorChange` and `AnchorStopError` come from card 9, `BranchAnchorStopped` from card 11, the move record from card 13, `integrateWeft` from card 14, `BeginAnchorChange` from card 15 and `ReanchorPair` from card 3, each ahead of its users.
- Fidelity: every Decision, D3, D5 and D7 included, is carried by some card. The uncarried and unlicensed pieces are F1, F7, F10 and F13.
- Re-sign arrows: present on every re-signed member (`readRecordedAnchor`, `pairLocation`, `create`, `createSteps`). No Edit card lists only non-test source paths while naming symbols.
- Test economy: no card plans a test whose behavior an existing test covers.
- Research steps: none found.
- PATTERN fit: no engine gains a `lyxcwd` import. `dobbyengine` is told its record paths through `hubgeom`. The departures found are F6 and F9.

## Rejected

- way-forward finding 4 (invalid own marker names no next step): D2 makes an invalid own marker "an error, as today's Bolt read is", and card 1 mirrors that existing error naming the file. The plan adds no new refusal here.
