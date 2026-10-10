# Webster and round-gate bug sweep: replans, batch checks, fork sandbox, shell wait, test re-runs

`webster-replan-state` · Task · bug, enhancement, webster, loom, burler, fabric · running · Webster

## Goal

- A `lyx loom goto` back to Plan after Webster started leaves webster state consistent, so Webster never hard-fails on a plan fingerprint mismatch (#466).
- A Plan-Review round after the plan is implemented is not dispatched as a fix round that edits the plan webster recorded (#465).
- Rebaseline logs and names the silent overview-frame fallback when the baseline copy has no parseable Card Index.
- `commitsNamedBy` classifies every `head_sha` prefix candidate in one `git cat-file --batch-check` call.
- `verify-nested-module` (`crossModuleCommands` in `internal/planparser/verifymodule.go`) splits a verify chain at `||` and `|` as well.
- `lyx webster record-batch` applies create-not-done only to the cards of the batch it records, and its refusal names batch and card consistently (#484).
- An implementer fork cannot reach the real hub's fabric or remotes: its stencil states the sandbox-hub rule, lyx refuses `lyx fabric` writes and pushes to the hub's own remotes from an implementer strand, and `lyx fabric add --help` says it pushes (#494).
- The fabric-reference check allows read-only commands (`--help`, reads of fabric files, `git remote get-url`, `git ls-remote`), and `recover-batch` accepts pathless findings whose recorded commands are all read-only (#493).
- When Master's wait stops waiting on a background shell, lyx stops or records the shell, and the shell-wait rule is the same for transcript-reported and Stop-payload-reported shells (#496).
- The round gate's failing-test re-run (`publishFailureCommand` in `internal/loomshed/publishfailure.go`) runs the failing test's top-level test, never a subtest alone, since subtests may share state (#497).
- `TestLoomResumeReconcilesTheHubConfigBeforeArming` in `internal/loomcli` is deterministic (#492).
- A dead batch whose recovery left commits gets a second recovery without the orchestrator (#498; the turn-end classification half landed in 7462c1dc5).

The merged notes keep their own Seen sections below.

## From `goto-plan-stale-webster`

Imported from [issue #466](https://github.com/Knatte18/loomyard/issues/466).

## What happened

A loom run completed the Plan, Batchifier and Webster rows, then the Publish row went stuck and the run routed back (`goto`) to `Plan-Bouncer` several times.
The Plan loop edited the plan, and `Batchifier` re-ran and finished (`done`).
The next `Webster` step then halted the run as `failed` with anomaly `producer-hard-failure`:

```
webster: on-disk plan fingerprint does not match this run's recorded state: on-disk plan fingerprint e5774b7e... does not match this run's recorded fingerprint c14834a3...; the plan changed since state.json was created; way forward: 00-overview.md changed and is never rebaselined; restore it with "lyx webster restore-plan", or 1) lyx webster reset --to start; 2) re-step the Webster row
```

The run had 49 history entries when it halted.

## Why this is abnormal

The loom's own routing sent the run back through Plan after Webster had already created `state.json`, and then re-ran Webster against that stale state.
The recorded fingerprint can only match if the plan is untouched, so a legitimate Plan-loop edit made the next Webster step fail by construction.
Neither the `goto` to Plan nor the re-run `Batchifier` row reset or rebaselined webster state, so the failure was foreseeable from the recipe's own wiring rather than an operator or task-code error.
The error text offers manual recovery (`lyx webster restore-plan`, or `lyx webster reset --to start` and re-step), which a driver could have applied itself.

## What to do

When the loom routes back to a row upstream of Webster after Webster state exists, reset webster state (equivalent to `lyx webster reset --to start`) as part of that route, or have the Webster row detect a changed plan produced by an upstream re-run and rebaseline instead of failing.
Alternatively, make the recipe validate at the `goto` that a Plan re-entry invalidates downstream webster state.

Provenance: task slug `shuttle-send`; notes: `loom-halt.md`.

## From `plan-review-after-implemented`

Imported from [issue #465](https://github.com/Knatte18/loomyard/issues/465).

A Plan-Review round ran over a plan that was already approved and implemented by webster.
The review gained findings, but the fixer's only write surface was the plan, and webster could not accept a new card on this run's record.
`lyx webster rebaseline --card=[11,12]` was refused:

```
webster: the edited plan changes the cards of a batch this run already begun: 00-overview.md changed and this run recorded no overview frame, so the change cannot be confined to its Card Index; it carries the plan's integration verify and is never rebaselined; way forward: restore 00-overview.md, or 1) lyx webster reset --to start; 2) lyx webster run
```

The parent stopped all fixes mid-round, so every finding ended deferred.
The fixer also spent its orientation designing a new card that was then thrown away.

Why this is abnormal: the loom dispatches a review round whose fixes cannot land.
A frozen plan (webster already begun) makes the fix round unwinnable, and the only advertised way forward (`lyx webster reset --to start`) discards the implemented work.
The run records no overview frame, so a legitimate edit to the overview's Card Index cannot be confined and is refused outright.

What to do:
- Skip the fix-round dispatch, or tell the round prompt up front that the plan is frozen, when the run's webster record has begun a batch.
- Record an overview frame when webster begins a batch, so a Card Index change can be confined and rebaselined.

Provenance: task `shuttle-send`; notes `burler-plan-r1.md`, `webster-refusal-rebaseline.md`.

## From `overview-frame-fallback-log`

## Seen

The PR-Gate review of `webster-record-batch` (PR #473) found that `recordedOverviewFrame` in `internal/websterengine` returns an empty frame without logging when the stored baseline copy has no parseable Card Index, so the reason a rebaseline refuses on an old record is invisible.

## Idea

Log the fallback at Info with the baseline path, and name it in the rebaseline refusal.

## From `head-sha-batch-check`

## Seen

The PR-Gate review of `webster-record-batch` (PR #473) found that `commitsNamedBy` in `internal/websterengine/gitwrap.go` runs one `git cat-file -t` per object a short `head_sha` prefix names, and a 4-character prefix in a large repository can name many.

## Idea

Classify every candidate in one `git cat-file --batch-check` call.

## From `verify-module-or-pipe`

## Seen

The PR-Gate review of `plan-gate-checks` (PR #475) found that `crossModuleCommands` in `internal/planparser/verifymodule.go` splits a verify chain only at `&&`, `;` and newlines, so a `go` command after `||` or a pipe is read as part of the previous segment and its module is never checked.

## Idea

Split at `||` and `|` too.


## From issue #476

Imported from [issue #476](https://github.com/Knatte18/loomyard/issues/476).

## What happened

A plan card required one exact sentence to be added to the `PATTERN-shuttle-stop` line in `PATTERN.md`, with nothing else changed.
With that sentence the line is 409 characters, but `internal/pattern`'s `MaxEntryLineChars` caps an entry line at 320, and `TestOverview_RealFilePassesCheck` enforces the cap.
The plan was approved anyway, so the card could not be satisfied as written.

## Why this is abnormal

The implementer had to reword the whole line.
The reviewer, checking against the card's text, raised the reword as a finding (F1) and proposed a fix that fails the plan's own verify.
That produced extra Webster-Burler rounds on a defect that was fixed in the plan, not in the code.
The plan gate (Plan-Write, Plan-Bouncer) has no check that exercises a card's verbatim PATTERN text against the entry cap.

## What to do

Add a plan-gate check: when a card prescribes verbatim text for a `PATTERN.md` entry line, compute the resulting line length and reject the card if it exceeds `MaxEntryLineChars`.
Alternatively, tell Plan-Write in its instructions to check verbatim PATTERN text against the cap before approving the card.

Provenance: task `shuttle-send`; notes `burler-webster-r1.md`, `loom-halt.md`.

## From `record-batch-card-scope`

Imported from [issue #484](https://github.com/Knatte18/loomyard/issues/484).

`lyx webster record-batch 01` refused with a failed batch.
The refusal names batch `01-headcontains-bounded-walk` but blames card `3-proc-group-kill`: `create-not-done/3-proc-group-kill[blocking]: Create target "plan:internal/proc#ConfigureGroupKill" still does not resolve`.
The report was archived under `_lyx/webster/reports/` and the way forward given was `lyx webster recover-batch 01`.

Why this looks abnormal: the failing card's slug differs from the batch being recorded, so either the create-not-done check evaluates cards outside the batch being recorded, or the batch and card numbering are mixed up.
It is also unclear whether the "still does not resolve" check resolves the `plan:` symbol against the right tree (the worktree head versus the card's merged result), since the symbol was later made to resolve by a follow-up commit.
If the check is correct and the card simply had not created the symbol, this is a task defect and can be closed.

What to do: confirm that `record-batch` only applies create-not-done to cards of the batch being recorded, and that the refusal message names the batch and card consistently.

Provenance: task `git-robustness`; notes: `webster-refusal-record-batch.md`.

## From `fork-real-remotes`

### Seen

On 2026-10-10, `daemon-wakeups` batch 09's implementer fork ran `lyx fabric clone/add --help`, read the real hub's `_lyx/fabric/origin.json` and ran `git ls-remote` on the weft repo.
The card 14 fork's `lyx fabric add idle-a/b/c` pushed branches `idle-a`, `idle-b`, `idle-c` to `Knatte18/loomyard` and `idle-*-weft` to `Knatte18/loomyard-weft`.
Recovery refused the batch with `needs_fresh`; the parent reset it and recorded a sandbox rule as a decision.

### Idea

An implementer fork cannot reach the real hub's fabric or remotes: its stencil states the sandbox-hub rule (local bare repos as origins), and lyx refuses `lyx fabric` writes and pushes to the hub's own remotes from an implementer strand, the way the hub-wide config refuses writes from a task worktree.


### From issue #494

Imported from [issue #494](https://github.com/Knatte18/loomyard/issues/494).

An agent following a "scratch hub" procedure cloned the real weft remote and ran `lyx fabric add` three times.
Each call pushed its code and weft branches to the real origin, leaving stray branches on both the code and weft repositories.
Nothing in `lyx fabric add` output or `--help` made clear that it pushes, so the scratch hub was not isolated.

Suggested fix: document the push in `lyx fabric add --help`, and offer a flag or setting for a local-only add, or a way to point a scratch hub at a local bare remote.

Provenance: task `daemon-wakeups`; note `09-orch-held-injection-log.md`.

## From `webster-fabric-ref-check`

Imported from [issue #493](https://github.com/Knatte18/loomyard/issues/493).

`lyx webster record-batch 09` refused: an implementer fork's batch failed on four "fabric-reference violation" findings.
The flagged commands were read-only: `lyx fabric clone --help`, `lyx fabric add --help`, `lyx reed up --help`, `cat _lyx/fabric/origin.json`, `git -C loomyard-weft remote get-url origin` and `git ls-remote --heads`.
`lyx webster recover-batch 09` then refused with `needs_fresh`, because the findings name no path and recovery cannot check them.
The run halted `blocked` (escalation-to-human) at Webster until a binary-change re-step let it continue.

This is abnormal: the check treats any command mentioning the fabric as one that "can rewrite run state", including `--help` invocations and plain reads.
A fork that only reads help text or a config file cannot rewrite anything, so the batch fails and halts the run for no real violation.

Suggested fix: classify fabric-referencing commands, and allow `--help`, `cat`/`ls` of fabric files and read-only git queries (`remote get-url`, `ls-remote`).
Also make `recover-batch` able to accept findings that carry no path when every recorded command is read-only.

Provenance: task `daemon-wakeups`; notes `loom-halt.md`, `webster-refusal-record-batch.md`, `webster-refusal-recover-batch.md`, `driver-restep-2a77897a307a6691.md`.

## From `webster-background-shell-wait`

Imported from [issue #496](https://github.com/Knatte18/loomyard/issues/496).

A `webster-background-shell` note reported that Master's wait stopped waiting on a still-running background shell at a turn end.
The wait counts the turn end after `background_shell_wait_min` (10 minutes) for a shell only the transcript reports, and at once for one the Stop payload reports when Master's output files exist.
lyx did not stop the shell; it ended only because shuttle removes Master's strand when the run finishes.
The run's final outcome was `done`.

This is abnormal because a background shell can outlive the decision to stop waiting, and its output is lost without any record beyond the note.
Suggested fix: stop or explicitly report the shell when the wait gives up, or make the shell-wait rule consistent between transcript-reported and Stop-payload-reported shells.

Provenance: task `daemon-wakeups`; note `webster-background-shell.md`.

## From `subtest-rerun-shared-state`

Imported from [issue #497](https://github.com/Knatte18/loomyard/issues/497).

Seen (publish-recovery, 2026-10-10): the Webster-Burler round gate failed three times on `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` with "probe run lock: ... .lyx/shed/loom-smoke-task/run.lock: no such file or directory".
`publishFailureCommand` (`internal/loomshed/publishfailure.go:88`, from 7648e8b27) re-runs each failing test from `publish-failure.yaml` alone, with a `-run` filter anchored on the subtest (`^TestSmokeBootstrapLifecycle$/^died_driver_...$`).
That subtest depends on the scratch dir and run lock that the earlier subtests' `loom start` bootstraps create, so run alone it always fails, while the whole test and the full tmux tier pass.
The fixer looped on a failure its own fix could not touch, and the run needed an investigation and a parent decision.

Workaround: a parent decision that the failure is a gate artifact, then resume.

Expected: the re-run names the failing test's top-level test (the first `-run` segment), never a subtest alone, since subtests may share state.

## From `flaky-resume-reconcile-test`

Imported from [issue #492](https://github.com/Knatte18/loomyard/issues/492).

The webster integration-suite triage reported `internal/loomcli.TestLoomResumeReconcilesTheHubConfigBeforeArming` as flaky: it failed the first verify run and passed or vanished on rerun, with no change in between.

This is abnormal because a test that fails nondeterministically blocks verify gates, and the run in question halted blocked at Publish with "verify failed after merging parent branch" (anomaly `escalation-to-human`).
The halt note and the verify log on disk do not show whether this test caused that failure, so the link is unconfirmed.

What to do: reproduce with repeated runs of `lyx gate test --tags integration ./internal/loomcli`, find the ordering, timing or shared-state dependency in the test, and make it deterministic.

Provenance: task `bolt-and-store`; notes `webster-verify-triage.md`, `loom-halt.md`.


## From issue #498

Imported from [issue #498](https://github.com/Knatte18/loomyard/issues/498).

Seen (test-suite-weight, 2026-10-10): `lyx webster record-batch 01` refused on create-not-done, and `lyx webster recover-batch 01` spawned a recovery strand.
The strand committed a working fix (f4c8a9103), then ended its turn with "Batch gate is running; I'll write the report once it finishes".
`internal/websterengine/classify.go:43-45` classifies a turn end with no report as terminal `dead/asking`, and `awaitTerminal` (`recoverbatch.go`) stops the strand, so the report was never written.
Once dead, `record-batch` refuses, `begin-batch` reports the batch exhausted, and Master's stencil tells Master to stop on `dead`; the run blocked at Webster, and a `lyx loom goto --to Webster` plus resume blocked again on "Batch 01 is dead".

Workaround: the orchestrator ran `lyx webster recover-batch 01` again by hand; `RecoverSpawnOrAttach` spawns a fresh cold recovery strand when the prior record is terminal dead.

Expected: a recovery strand waiting on its own background gate is not dead at a turn end (wait for the shell, or its stencil forbids ending the turn before the report), and a dead batch whose recovery left commits gets a second recovery from Master or the Webster row without the orchestrator.
