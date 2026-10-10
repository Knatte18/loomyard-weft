# Decision record: webster-replan-state

## Goal

Webster stays consistent across replans, its batch and fabric checks judge only what they should, implementer forks stay sandboxed, the shell wait leaves nothing unrecorded, and the round gate re-runs whole tests.
Twelve defects from the board entry (#466, #465, #476, #484, #494, #493, #496, #497, #492, #498, plus the overview-frame log, the batch-check call and the verify-chain split) are fixed in one sweep, each with its test and its doc in the same commit.

## Scope

In:

- A `lyx loom goto` back to a row before Webster, after Webster recorded state, never hard-fails the next Webster step on a plan fingerprint mismatch (#466).
- A Plan-Review round over a plan webster has begun tells the reviewer and fixer which cards are frozen, refuses an edit to a done card at the plan gate, and lands follow-up cards through webster (#465).
- `recordedOverviewFrame` logs its empty-frame fallback and the rebaseline refusal names it.
- `commitsNamedBy` classifies every candidate in one `git cat-file --batch-check` call.
- `verify-nested-module` and `verify-module-wide` split a chain at `||` and `|` too.
- `record-batch`'s done-checks are pinned to the batch's own cards and its refusal names batch and card consistently (#484).
- An implementer fork or recovery strand cannot drive the real hub's fabric: the stencil states the sandbox rule, `lyx fabric`'s mutating verbs refuse from such a strand, and `lyx fabric add --help` says it pushes (#494).
- The fabric-reference audit treats a read-only command as policy, and recover-batch accepts a failed batch whose pathless findings are all read-only (#493).
- One background-shell wait rule for transcript-reported and payload-reported shells, and a record of every shell a run ends over (#496).
- The round gate's failing-test re-run names the top-level test only (#497).
- `TestLoomResumeReconcilesTheHubConfigBeforeArming` is deterministic (#492).
- A dead batch whose recovery left commits gets one more recovery from Master, bounded (#498).
- A card that prescribes a verbatim `PATTERN.md` entry line over `MaxEntryLineChars` fails the plan gate (#476).

Out:

- Any Go-side route from a blocked row to a fixer (publish-recovery decided the hub runs the goto).
- Killing an individual background shell inside a provider session; lyx has no pid for it.
- A local-only `lyx fabric add` flag or a scratch-hub remote switch: refusing the verb from the sandboxed roles covers the seen failure.
- Making quarry honour build constraints (board note `quarry-build-tag-ambiguous`).
- The overview size cap at Publish (board note `pattern-overview-headroom`) and the per-attempt verify log (board note `verify-log-overwritten`).
- Bounce-budget ordering (board note `bounce-budget-after-approved`).

## Decisions

### D1. The Webster row rebaselines a changed plan on entry instead of failing

Decision: `websterengine.RunOptions` gains a mode the loom's Webster producer sets and `lyx webster run` never sets: on a fingerprint mismatch at entry, Run calls `Rebaseline` with `Cards` set to the number of every changed card file (`changedPlanFiles`), through the same deps `lyx webster rebaseline` builds.
An accepted rebaseline is logged, carried as a `RunResult` warning and the run continues over the kept batches and the re-batched tail.
A refused one, and any mismatch `lyx webster run` itself meets, keeps today's `ErrFingerprintMismatch` or `ErrRebaselineCardSetChanged` text; the loom's producer maps both to `Stuck` with the refusal as the reason, never to a returned error, so the run blocks resumable instead of failing with `producer-hard-failure`.
Bound: the auto-rebaseline accepts exactly what an operator naming every changed card with `--card` gets: a done batch's card edit, a begun batch's card-set change, a failed batch with uncheckable findings and an overview change outside its Card Index are still refused; a card of an in-flight batch, which exists at row entry only after a crash, is accepted as an amendment and re-run by the next recovery.
Rationale: the Plan loop edits the plan in place after a goto to Plan-Bouncer, Plan-Burler or Batchifier, and only a goto to Plan-Write rotates the plan and archives webster's record (PATTERN-plan-generation); every other replan reaches Webster with a record the operator wants kept, and the advertised way forward was always a rebaseline.

### D2. A Plan-Review round over a begun webster run knows what is frozen

Decision, three parts:

- The Plan-Review rubric and the Plan-Burler fixer prompt gain an optional `webster_record` marker, filled by Go the way `publish_failure` is: `none` with no webster record, else the begun batches, the done cards, and the rule that a done card is frozen and a finding against it lands as a follow-up card after the last begun batch with its Card Index line in `00-overview.md`.
- The plan gate (`loomshed.ValidatePlan`, already told webster's done cards) gains a blocking `done-card-edited` finding: a done card whose file hash differs from the hash webster recorded at begin, computed in `websterengine` beside `DoneCards`, naming the card and the follow-up landing.
  Bound: the check reads webster's record and card bytes, never parses the plan; with no record or no recorded hashes it reports nothing.
- The follow-up cards reach webster through D1's rebaseline at the Webster row.

Rationale: the fix round is winnable once the fixer knows the write surface, and the gate turns an unwinnable edit into a finding at the round instead of a refusal at Webster.
The fix-round dispatch itself is not skipped: findings against unbegun cards stay fixable.

### D3. The overview-frame fallback is logged and named

Decision: `recordedOverviewFrame` logs at Info, with the baseline copy's path, when the copy is absent or has no parseable Card Index, and `overviewRefusal`'s no-frame text names which of the two it was and the path.
The refusal-spec row `rebaseline overview changed` carries the new wording.

### D4. One batch-check call per prefix

Decision: `commitsNamedBy` feeds every object name `rev-parse --disambiguate` lists to one `git cat-file --batch-check` run and keeps the lines typed `commit`; a `missing` line is an error.
The call goes through `gitexec`'s checked form with stdin, added there if it has none (PATTERN-gitexec-checked-call).

### D5. One chain splitter for the verify checks

Decision: `planparser` gets one chain splitter that both `crossModuleCommands` and `moduleWideCommands` use, splitting at `&&`, `||`, `;`, `|` and newline, with `||` never read as two pipes.
Rationale: both checks had the same gap; one splitter keeps them in step.

### D6. Done-checks stay on the batch's cards, and the refusal says which batch holds the card

Decision: a table row pins that `postBatchChecks` hands `DoneChecks` the batch's own cards only, for a multi-card batch beside unbegun cards.
`failCardNotDone`'s reasons open with the batch's id and its card list (`batch 01-headcontains-bounded-walk holds cards 1-…, 2-…, 3-proc-group-kill`), so a finding keyed on a later card of the batch reads as the batch's own.
Rationale: the code already scopes the checks; the seen refusal named a third card of a three-card batch, and the text gave no way to see that.

### D7. An implementer fork or recovery strand cannot drive fabric

Decision, three parts:

- `webster-body-implementer.md`, shared by the fork and the recovery prompt, states the sandbox rule: never clone the hub's remotes, never run a mutating `lyx fabric` verb; a scratch hub is a Go test fixture over local bare repositories (`hubforge`), and lyx refuses the verbs from your strand.
- `fabriccli` refuses its mutating verbs (`clone`, `add`, `remove`, `checkout`, `reconcile`, `prune --apply`, `cleanup --apply`, `commit`, `push`, `pull`, `sync`, `merge`, `merge-in`, `merge-stage`, `unwire`, `shortname`) from a session whose `LYX_STRAND_NAME` parses with a slug and a role other than the driver's (`agentname.RoleDriver`) or the conflict resolver's (`mergeresolve.ConflictRole`), with the way forward naming the sandbox rule and the orch as the one who runs fabric verbs.
  `list`, `pairs`, `status`, `diff` and every `--help` pass.
  Bound: like `configcli`'s hub-wide guard it reads the strand name and nothing else, so it stops a mistaken call, not a fork that clears the variable; a raw `git push` is outside lyx and stays the stencil's and the audit's.
- `lyx fabric add`'s long help states that it pushes the new code and records branches to origin.

Rationale: the driver and the conflict resolver are the only slug-bearing roles whose stencils name fabric verbs; every other task-session role has no business in fabric, so deny-by-default with that allowlist.

### D8. A read-only fabric reference is a warning, not a failed batch

Decision: `fabricengine.IsReadOnlyCommand` becomes the one classifier, closed and static: every segment of the command (split as D5 splits, and refused on any `>` redirection) is a reader from today's list, a `lyx` invocation carrying `--help` or `-h` or whose fabric verb is `list`, `pairs`, `status` or `diff`, or a `git [-C <dir>]` read (`remote get-url`, `remote -v`, `remote show`, `ls-remote`, `rev-parse`, `log`, `show`, `status`, `diff`, `branch` with a list flag, `cat-file`, `config --get`).
`ClassifyViolation` classes a fabric reference whose command the classifier accepts as policy, so the batch records an audit warning and passes.
`RecoverSpawnOrAttach` accepts in line a terminal failed batch whose `Uncheckable` entries are all pathless fabric references with read-only commands, under `AcceptBatchFabricReference`'s evidence rule (HEAD at the batch start, or the start an ancestor and the tree clean), recording the warnings and spawning the recovery, so `needs_fresh` is reached only for a command the classifier rejects.
Bound: the classifier sees static shape only; a command behind `bash -c` or built from a variable keeps today's correctness class.

### D9. One shell-wait rule, keyed on the output files, and every shell a run ends over is recorded

Decision: shuttle's wait no longer distinguishes a transcript-reported from a payload-reported shell.
A turn end with every output file present counts at once, gated or not, whatever unawaited shells are outstanding; a turn end with an output file missing waits on an unawaited shell of either signal until the run's own deadline and the liveness check.
`background_shell_wait_min` stays in `shuttle.yaml` as the threshold after which an outstanding shell is logged once at Warn and shown in the wait marker; it expires nothing, and `ShellWaitExpires` follows the same rule so webster's recovery classification agrees.
`Result.ExpiredShells` becomes a record of each shell the run ended over: label, id, signal and how long it was outstanding; webster's summary section and friction note render the record, and the strand removal that ends the run logs that it ended those shells.
Bound: lyx kills no shell by itself, since the provider reports no pid; a shell detached from the pane's process tree survives the removal, and the record says the removal ended the shells it could.
Rationale: 7462c1dc5 made a payload shell never expire because a gate behind a slot wait can outlast ten minutes; the transcript fallback is the same work reported through a weaker signal, and the output files, not the signal, say whether the agent is finished.

### D10. The round gate re-runs the top-level test

Decision: `publishFailureCommand` anchors `-run` on the first `/` segment only (`^TestX$`) and collapses the record's tests to one step per package and top-level test; `PublishFailureNote` keeps the full subtest path.

### D11. The resume-reconcile test is made deterministic by reproduction

Decision: the plan's card reproduces `TestLoomResumeReconcilesTheHubConfigBeforeArming` under repeated runs of the package's integration tier, names the dependency it finds and removes it; the fix is not predicted here.
Candidates to rule out first: the 30 s and 60 s subprocess timeouts of a cold `lyx` under a loaded hub, `sharedLyxBinary`'s `os.MkdirTemp("", …)` outside the test's temp dir, and the reconcile commit's detached push on the `loom status` call, which runs without `FABRIC_SKIP_PUSH`.

### D12. A dead batch whose recovery left commits gets one more recovery

Decision: `BatchState` gains `Recoveries`, the count of recovery strands spawned for the batch.
`RecoverSpawnOrAttach` over a terminal `dead` prior whose recovery left commits (HEAD past the batch's `StartSHA`) spawns a fresh recovery while `Recoveries < 2`; a third is refused with `recovery_exhausted`, naming `lyx webster reset --to batch-start --batch NN` then `recover-batch`, or `reset --to start`.
The recover-batch envelope carries `recoveries` and `recovery_retry: true` when a retry is available, and the Master stencil's dead rung says: `status: dead` with `recovery_retry: true` runs `recover-batch <NN>` once more, backgrounded; otherwise the run ends stuck as today.
Bound: at most two recoveries per batch, and only when the prior one committed.
Rationale: Master is the caller that already loops on `card_amended`; the Webster row stays Go that runs Master, and the orchestrator is out of the loop.

### D13. A verbatim `PATTERN.md` entry line is measured at the plan gate

Decision: `planparser` gains the blocking check `pattern-entry-line-cap`: a line inside a card's fenced block or inline span that starts with `` - `PATTERN- `` and exceeds `pattern.MaxEntryLineChars` runes fails the card, naming the entry and the length.
`loom-template-plan.md` tells the plan writer that a prescribed addition to an entry line must leave it under the cap.
Bound: the check measures verbatim lines only; a prose instruction to append a sentence is the stencil's rule.

## Constraints

- PATTERN-refusal-way-forward: every new or reworded refusal in webster, shed or loom (D1's Stuck reason, D3, D6, D12's `recovery_exhausted`) names its way forward, lands its spec row in the same commit and is reached by a test; `fabriccli`'s D7 refusal is outside the bound modules and still names a way forward.
- PATTERN-no-denied-recovery: no way forward names a denied form; D12's names `lyx webster reset`.
- PATTERN-friction-capture: D7's and D12's refusals write friction notes as every refusal does.
- PATTERN-sole-parsers: D2's hash compare and D13's line scan read bytes and raw card text, never a second parse of the plan.
- PATTERN-plan-generation: D1 changes nothing about rotation or archiving; a goto to Plan-Write still starts a new webster run.
- PATTERN-batcher-registry: D1's partition replacement is a rebaseline, the one allowed replacer.
- PATTERN-review-round: D2 skips no fix round and changes no convergence rule.
- PATTERN-transient-stop: D1's Stuck carries no transient mark.
- PATTERN-agent-name: D7's role allowlist cites the owning modules' constants.
- PATTERN-shuttle-provider-seam: D9 changes `shuttleengine` only; `claudeengine`'s event parsing is untouched.
- PATTERN-wait-mark-display-only: D9's marker change is display only.
- PATTERN-gitexec-checked-call: D4 adds no raw call site.
- PATTERN-gate-self-check-parity: D2's and D13's checks run in the gate closure and in `lyx loom validate-plan` through one function.
- PATTERN-stencil-ownership and PATTERN-producer-pointer-rule: D2's and D7's stencil text points at the rules' owners and duplicates no contract.
- PATTERN-test-economy, PATTERN-test-speed, PATTERN-test-isolation: new cases are rows in existing tables at the public surface; D11 stays in the integration tier.
- PATTERN-documentation-lifecycle and PATTERN-comment-line-breaks: `doc.go` of `websterengine`, `shuttleengine`, `fabriccli`, `loomshed` and `planparser` change in the same commits.
- Test budget: a card that pushes `websterengine`, `planparser`, `loomshed`, `fabriccli`, `fabricengine`, `shuttleengine` or `loomcli` past its row lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.

## Auto-mode assumptions

- #476 is in scope: the board entry lists it among the task's issues, which close at landing, and its body carries the section.
- D1 rebaselines at the Webster row rather than resetting webster state at the goto, since a reset discards implemented work.
- D7 keys the fabric refusal on the strand's role with the driver and conflict roles allowed, not on comparing remote URLs.
- D9 unifies on the payload rule, not the ten-minute expiry.
- D12 keeps Master as the caller of the second recovery.

## Open risks

- D9: a transcript-reported shell that is stale holds a run until its own timeout, where it expired after ten minutes before.
- D1: after a crash, an edit to a card of the in-flight batch is adopted as an amendment on row entry without an operator naming it.
- D7 and D8 guard static shape and the exported strand name; a fork that clears the variable or hides a command behind `bash -c` is caught by review, not lyx.
- D11 may not reproduce on a quiet machine; the card then documents the attempt and the timeouts it raised.

## Acceptance criteria

- A run with webster state, moved by goto to Plan-Bouncer, whose Plan loop adds a follow-up card after the last begun batch, continues at Webster over the kept batches with the new card batched, and the step records a warning naming the rebaseline.
- The same run whose Plan loop edits a done card blocks at Webster as Stuck with the rebaseline refusal, never `failed`; `lyx webster run` on that worktree still refuses with the fingerprint text.
- The plan gate reports `done-card-edited` for an edited done card and nothing for an unbegun card, with and without a webster record.
- The Plan-Review rubric renders `none` without a webster record and the begun batches, done cards and follow-up rule with one.
- `rebaseline` over a record with no frame hash and a baseline copy without a Card Index logs the fallback with the path and refuses naming it.
- `commitsNamedBy` with a prefix naming a commit, a blob and a tree runs one batch-check and returns the commit; a `missing` line is an error.
- `verify-nested-module` and `verify-module-wide` find a `go` command after `||` and after `|`, and `a || b` is one `||`.
- A three-card batch beside unbegun cards: done-checks see the three cards only, and the failure reasons open with the batch id and its card list.
- `lyx fabric add`, `clone`, `push` and `sync` refuse under a webster or recovery strand name and run under the driver and conflict names and under no name; `list`, `pairs` and `add --help` pass under every name.
- The classifier accepts `lyx fabric add --help`, `cat _lyx/fabric/origin.json`, `git -C x remote get-url origin`, `git ls-remote --heads` and a pipe of readers, and rejects `lyx fabric add x`, `cat a > b` and `git push`.
- A fork transcript with only read-only fabric references records-batch done with audit warnings; one with `lyx fabric add` fails as today.
- recover-batch over a failed batch whose uncheckable entries are all read-only fabric references spawns a recovery and records the warnings; a non-read-only entry still refuses `needs_fresh`.
- A gated run with output files present and a transcript-reported shell outstanding finishes at once with the shell in the record; with an output file missing, both signals wait past `background_shell_wait_min` and log once; `ShellWaitExpires` agrees for a recovery strand.
- The run's end logs the shells the strand removal ended, and webster's summary lists them with signal and duration.
- `publishFailureCommand` for `TestA/sub_1` and `TestA/sub_2` in one package emits one `-run '^TestA$'` step; the note keeps both subtest paths.
- Repeated integration runs of `internal/loomcli` pass with the resume-reconcile test included.
- A dead batch with commits past its start gets a second recovery and `recovery_retry: true`; a dead batch without commits, and a third attempt, refuse with the named way forward; the Master stencil's dead rung names the retry.
- A card whose fenced block holds a `PATTERN-` entry line over the cap fails `pattern-entry-line-cap`; one under the cap passes.

## Notes for the plan writer

- D1: `internal/websterengine/runlevel.go` (the fingerprint case of Run), `rebaseline.go`, `internal/webstercli/rebaseline.go` for the deps constructor, `internal/shedadapters/webster.go` for the Stuck mapping, `internal/loomshed/webster.go` for the row.
- D2: `internal/shedadapters/rubric.go` and `bouncer.go` for the marker plumbing, `contracts/stencils/loom/loom-rubric-plan-review.md`, `contracts/stencils/burler/burler-step-3-fix.md`, `internal/loomshed/gates.go` (`ValidatePlan`), `websterengine.DoneCards` and `BatchState.CardHashes`.
- D3 and D4: `internal/websterengine/rebaseline.go`, `gitwrap.go`, `internal/gitexec`.
- D5 and D13: `internal/planparser/verifymodule.go`, `fabricref.go`'s `codeSpans` for the span walk, `internal/pattern/check.go` for the cap.
- D6: `internal/websterengine/recordbatch.go` (`postBatchChecks`, `failCardNotDone`).
- D7: `internal/fabriccli/fabric.go` and its persistent pre-run, `internal/configcli/hubwide.go` as the model, `contracts/stencils/webster/webster-body-implementer.md`.
- D8: `internal/fabricengine/refscanner.go`, `internal/websterengine/audit.go` (`ClassifyViolation`), `auditledger.go`, `recoverbatch.go`.
- D9: `internal/shuttleengine/wait.go` (`expiredTurnEnd`, `ShellWaitExpires`), `run.go` (`Result.ExpiredShells`), `internal/websterengine/summary.go`, `triage.go`, `runlevel.go`, `strand.go`.
- D10: `internal/loomshed/publishfailure.go`.
- D11: `internal/loomcli/smoke_prebootstrap_integration_test.go`, `smoke_helpers_test.go`.
- D12: `internal/websterengine/state.go`, `recoverbatch.go`, `internal/webstercli/recoverbatch.go` for the envelope, `contracts/stencils/webster/webster-template-master.md`.
- Spec rows: `contracts/specs/refusal-spec.md`'s webster section.
