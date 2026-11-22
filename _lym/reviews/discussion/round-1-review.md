---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:42
    summary: "D2's hold rests on a false premise. `lym batten pause` sets only batten's own pause_requested, so the inner loom or Webster run keeps making rows, batch commits and Parent-Align merge-ins. The one hold that does stop the inner run, a `lym tender pause` entry, also holds every creel delivery into the writer's pane, so the D2 ask cannot arrive under it. The record pins neither behaviour."
    fix: "Rewrite D2's first paragraph and its Rationale so the order is: (1) ask the live writer by `lym creel post --expect-reply <strand>` for the note's first three parts (done, decisions, left) while no pause covers the slug; (2) once it replies, hold the run with `lym tender pause <slug>`; (3) the orch itself then records the fourth part, the task branch's tip SHA and its merge base with the old parent branch, read after the hold. Pin each fact with its evidence: `internal/battenshed/innerrun.go:400-401` and `:782-783` (batten pauses only at its own row boundary), `internal/webstercli/batchpause.go:33-54` and `internal/loomcli/pausehold.go` (the inner run honours only the pause record), and `internal/creelengine/doc.go:258` (a covering pause holds every delivery into the pane). In D7, replace `lym batten run <slug>` with `lym tender resume <slug>` as the lift of the pause. Update D4 step 1, D3's opening, and the D2 and D7 acceptance bullets to match."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:126
    summary: "The Constraints bullet on `--remove`, the owner bullet's claim that the archive tags are on origin, and D4 step 2's use of `archive_tags` misstate the cancel. In production `DiscardTaskBranch` and the records archive hand their remote half to a detached teardown child and return before any tag reaches origin. `archive_tags` then names only the local tip's tag, pushed later. It omits origin's tip tag when the tips diverge, and it is empty when only origin held the branch, though the child still archives origin's tip. A failed push is only mailed to the orch as a remote-teardown notice. So the carrier can name a tag origin never gets, or name none where one exists."
    fix: "Rewrite the Constraints bullet to say: with `--remove`, cancel creates the archive tags locally and a detached teardown child pushes them and deletes the branches on origin after the command returns; a failed push mails the orch a remote-teardown notice naming the manual tag push. Pin it at `internal/fabricengine/taskbranchdiscard.go:24-41` and `:79-97`, `internal/fabricengine/remoteteardown.go:274-287` and `:319-333`, and `remoteteardown.go:184-207`. In D4 step 2, drop the rule that an empty `archive_tags` means no tag. Derive the code tag name from the tip SHA recorded after the hold (F1), as `archive/<slug>/<first 12 hex of the tip>`. Before step 3, have the orch confirm that tag on origin with `git -C <code repo> ls-remote origin refs/tags/archive/<slug>/*`. On a remote-teardown failure notice, take its way forward before step 3. Make the owner bullet's 'the archive tags are on origin' depend on that check."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:71
    summary: "D4 runs the irreversible cancel before the board write that moves the entry, and nothing checks first that the new parent takes a member. The board refuses a project held by a run (`internal/boardengine/project.go:99`), so a beam whose pawl run holds its entry refuses the `project` change. A beam past `lym beam ready` never starts a member filed after it (`contracts/orchrole/role-beams.md:61`). Either case leaves the task cancelled, a draft under its old parent, with nothing moved."
    fix: "Add a step to D4 before the cancel: the orch checks that the new parent can take a member. A beam must be neither done, nor held by its run (status past its hold-free rows, `internal/boardengine/runlock.go:44` and `internal/boardengine/project.go:99`), nor past `lym beam ready`. The root always qualifies. When the check fails, the move stops before the cancel: lift the pause and tell the operator. Add the check to the D4 acceptance bullet."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:149
    summary: "D5's re-run of the same slug is refused while the records repository keeps a stale tracking ref of the deleted records branch. The record names the workaround only under Open risks; no step of D4 runs it and no acceptance criterion carries it into `role-beams.md`. So the written procedure stalls at release until `stale-records-ref-blocks-rerun` lands, and the orch following it has no step for the refusal."
    fix: "Add to D4 step 4, before the release: while the entry `stale-records-ref-blocks-rerun` has not landed, the orch runs a pruning fetch of the records repository, `git -C <records repo> fetch --prune origin`. Cite that entry's body as the evidence (dobby's refusal `the code tip ... is not on origin yet`, cleared by the prune). Name that step in the D4 acceptance bullet. Replace the Open risks line with a pointer to the step."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:73
    summary: "D4 step 2 re-runs a cancel that 'reports a partial finish', and D7 stops the move and lifts the pause when cancel 'refuses'. The cancel reports neither by name: every failure is an error envelope (`internal/battencli/cancelcmd.go:89-92`, `:122-125`). Errors raised after `WriteIntent` and `EndForCancel` come after the run's processes have ended (`internal/battencli/cancel.go:372-409`, `:529-566`). An orch that reads one of those as a refusal lifts the pause on a half-cancelled run."
    fix: "In D4 step 2, define the two cases by the cancel's output. A partial finish is either an ok envelope with `end_only: true` or a non-empty `way_forward`, or an error whose way forward names the cancel itself to run again (`then run \"lym batten cancel <slug> --remove\" again`); the orch takes the fix it names and runs the cancel again. A refusal is an error whose way forward names another step, such as finishing the landing with `lym batten run <slug>` or `lym adopt`. D7 applies only to a refusal. Point D7 at that definition."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:77
    summary: "For a discussed entry, D4 step 3 adds 'the section' under the stored record's `## Notes for the plan writer`. The body variant names that section `## Move note`, an H2. The discussion spec admits no H2 sections besides its listed ones (`_bolt/_lym/specs/loom/loom-discussion-spec.md:13-25`), and the fresh run seeds that stored record as its own record. An H2 carrier, or a note whose four parts are H2 headings, makes the record malformed."
    fix: "In D4 step 3's stored-record branch, state that the carrier is an H3 `### Move note` under `## Notes for the plan writer`. Its four parts go in as bullets or H4 headings, never as H2. Add `## Notes for the plan writer` as the record's last section when the record lacks it. Keep `## Move note` for the body branch only."
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:119
    summary: "D8's glossary line defines the move note as what a cancelled run's writer learned, but under D3 the orch writes the note when no writer is live."
    fix: "Reword the glossary line in D8 to: `- **move note**: what a cancelled run learned, written for the fresh run that replaces it. Never: hand-back, handoff.`"
---

### [BLOCKING] The pause D2 uses does not hold the inner run, and the pause that does also holds the ask

**Location:** `_lym/discussion/decision-record.md:42` (D2), with D4 step 1 and D7.

**Issue:** `lym batten pause` sets `pause_requested` on batten's own status.
Only batten's Run-Shed and Land producers read it: Run-Shed ends its check and batten pauses at its own row boundary (`internal/battenshed/innerrun.go:400-401`, `:782-783`; verb text at `internal/battencli/cli.go:379-381`).
The inner loom and Webster run in the task worktree reads only the pause record that `lym tender pause` writes (`internal/loomcli/pausehold.go`, `internal/webstercli/batchpause.go:33-54`).
So Webster batches keep committing and Parent-Align can merge the parent in while the note is written, and the note's tip SHA and merge base drift from what the cancel archives.
The study the record cites already says this: "A pause stops a new `batten run` pass but not a driver already running" (`docs/research/loom/reparent-running-task-2026-11-12.md`, section 2, row 6).
A `lym tender pause <slug>` does hold the inner run, but "a pause that covers a strand holds every delivery into its pane" (`internal/creelengine/doc.go:258`), so the D2 creel ask would not reach the writer under it.
Neither existing hold gives both, and D7's lift (`lym batten run`) fits only the hold that does not work.
All four lenses found this independently.

### [BLOCKING] Cancel returns before its archive tags are on origin, and `archive_tags` does not list what the carrier needs

**Location:** `_lym/discussion/decision-record.md:126` (Constraints), with D4 step 2 and the owner Constraint.

**Issue:** `DiscardTaskBranch` takes its teardown hold with hand-off set, so outside a test binary a detached child runs the remote half after the command returns (`internal/fabricengine/remoteteardown.go:274-287`, `childHold.finish` at `:319-333`).
The result then carries only the local tip's tag, created but not pushed (`internal/fabricengine/taskbranchdiscard.go:79-97`).
When local and origin tips diverge, the child archives both and only one is reported (`taskbranchdiscard.go:105-134`).
When only origin holds the branch, `archive_tags` is empty though the child archives origin's tip.
A failed push is only logged and mailed to the orch (`remoteteardown.go:184-207`).
The record pins this behaviour to `cancel.go` and `cancelcmd.go`, but it lives one hop further, in fabricengine.
As written, the carrier can name a tag origin never receives, or drop a tag that exists.
D6's read path, the core of the Goal, then fails on the new owner machine.

### [MEDIUM] The cancel runs before anything checks that the new parent takes a member

**Location:** `_lym/discussion/decision-record.md:71` (D4 step 2).

**Issue:** `checkProjectField` refuses a project that a run holds (`internal/boardengine/project.go:99`).
A beam's pawl run holds its entry from Finalize on (`internal/boardengine/runlock.go:41-62`), and a member filed after `lym beam ready` never starts (`contracts/orchrole/role-beams.md:61`).
D4 cancels first and writes `project` second, so a move into such a beam leaves the task cancelled under its old parent with nothing moved.

### [MEDIUM] The stale-ref workaround stays in Open risks instead of the procedure

**Location:** `_lym/discussion/decision-record.md:149` (Open risks), with D4 and D5.

**Issue:** The fact and its workaround are known, with evidence in the `stale-records-ref-blocks-rerun` entry's body, so this is no deferred research question.
But no Decision puts the pruning fetch into the procedure `role-beams.md` will hold.
The orch would release the moved entry and meet dobby's refusal with no written step.
The orch role allows git through `git -C` (`contracts/orchrole/orch-role.md:426`), so the step fits the procedure.

### [MEDIUM] Partial finish and refusal are not told apart

**Location:** `_lym/discussion/decision-record.md:73` (D4 step 2), with D7.

**Issue:** The cancel's only marked partial finish is `end_only`.
Every other failure, including the run-phase and clear-phase errors after the processes ended, comes back as the same error envelope as a refusal.
Each failure names its own way forward.
D4 and D7 give the orch opposite actions, run again versus lift and stop, without saying which output selects which.

### [MEDIUM] An H2 move-note section in a stored record breaks the record format

**Location:** `_lym/discussion/decision-record.md:77` (D4 step 3).

**Issue:** The stored record is seeded as the fresh run's own record and must keep the spec's fixed H2 set (`loom-discussion-spec.md:13-25`).
D4 step 3 has the orch add 'the section', which the body branch names `## Move note`, under `## Notes for the plan writer`.

### [NIT] The glossary line names the writer, but D3 lets the orch write the note

**Location:** `_lym/discussion/decision-record.md:119` (D8).

**Issue:** "What a cancelled run's writer learned" excludes the D3 case, where the orch writes the note.

## Rejected

- bound-breaker's LOW on the records archive tag's origin namespace: on a one-repository hub, the records store's refspecs map origin's namespaced tags back to `refs/tags/archive/<slug>/…` on fetch (`PATTERN-single-repo-hub`), so the pattern the carrier names holds in the records repository of any machine that fetches.
- bound-breaker's note that D1's bound is overstated for the `--record-file` path: `StoreDecisionRecord` bypasses the lock only for the record, flag and stamp, and the `project` change in the same upsert still needs the cleared status, so the bound is correct as stated.
- Handler check on D3's use of the task branch log: the orch may run git through `git -C` (`contracts/orchrole/orch-role.md:426`), so the step is within its role.
- Handler check of the Constraints' PATTERN ids: `PATTERN-owner-machine`, `PATTERN-stencil-ownership` and `PATTERN-heddle-instructions` exist and say what the record cites, and `Owners.Effective` reads a task's owner through its parent (`internal/boardengine/owners.go:147-163`); no finding.
- Handler check of Scope and Constraints for deliberation to relocate: none found; each line carries a requirement or a pinned fact the plan writer needs.
