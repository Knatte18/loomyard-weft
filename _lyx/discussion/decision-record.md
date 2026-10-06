# Decision record — bugfix-webster

## Goal

Fix the Webster and plan-validity bugs the board entry `bugfix-webster` lists, so that a plan whose card order is not executable never reaches Webster, and so that each reported Webster, reset and Publish stop either cannot recur or names a way forward that works the first time.
Each fix carries a regression test that drives the real CLI wiring, not only fakes.

## Scope

In:

- `webster-delete-ordering` (and #401's first half): the Plan-Gate refuses a backward dependency between cards; a cycle is refused through it.
- `webster-delete-already-gone`: a **Delete:** target already gone at dispatch no longer blocks `begin-batch`.
- #401's second half: a `delete-not-done` failure whose cause is a later card still referencing the target names that cause and a way forward that is not a looping `recover-batch`.
- #402: `begin-batch` handles a report left with no begin-batch record itself, instead of pointing at `record-batch`.
- #403: `rebaseline --card` accepts an edit to a card of a failed or dead batch.
- #404: `webster reset` leaves the remote task branch consistent with the rewound local one, and Publish's push-rejected halt says what it tried and how to resume.
- `webster-background-shell-wait` (#397): the friction note states what happened to the shell and what the run did next.
- `webster-batch-validation-speed`: drop `begin-batch`'s per-batch model assertion.

Out:

- Webster dropping `SequenceBatches` and running batches in card-number order, and the rewrite of `loom-plan-spec.md`'s "DAG of intent, not an execution order" prose: both stay with the board note `webster-delete-ordering`.
- Incremental per-batch validation: quarry#34 is closed and the bump to quarry v0.2.1 already landed (`beea26e62`).
- Widening `lyx webster reset` with further targets (`webster-reset-targets` note).
- Any new verb, flag or config key.

## Decisions

### D1 — Plan-Gate refuses a card that uses a later card's target

Decision: a new blocking plan check refuses a card whose `Uses:` names an entry that appears in the flat target list (`Targets`, every type label including **Delete:**) of a card with a higher number.
The match rule is exactly `deriveEdges`' producer-before-consumer rule in `internal/websterengine/sequence.go`: exact string equality after trimming, empty entries ignored, ref kind not consulted.
It is a pure card-field check in `internal/planparser`, part of the format set, so it runs wherever that set runs: the Plan-Gate, the Rework-Plan-Gate, `lyx webster validate`, run entry and `begin-batch`'s dispatch validation.
At dispatch it compares only cards not yet completed (the existing dispatch scoping).
The finding names both cards and the shared entry, and its text tells the writer to move the work or reorder the cards so the producer comes first.

Rationale: a valid plan is executable strictly in card-number order (operator rule, 2026-10-06).
`deriveEdges`' other edge kind, two cards sharing a target, always points from the lower number to the higher, so once no `Uses` edge points backward every derived edge points forward.
A cycle needs a backward edge, so no separate cycle check exists, and `SequenceBatches` then returns the declared order unchanged (its documented no-op property): Webster already runs gated plans in card order without its sequencing being touched in this task.
Mirroring `deriveEdges` exactly, not a narrower rule, is what guarantees that: any edge the gate let through backward would still be reordered by Webster.

Bound: this refuses plans that pass today; the bound is that it refuses only a `Uses`/later-target match that would already make Webster reorder the cards.

### D2 — Plan-Gate refuses a delete that runs before a later card removes a reference to it

Decision: a new blocking plan check, in `internal/planglyph` because it reads the tree: for every **Delete:** target of card N that is a member or unit glyph, it searches the code each card numbered above N names under `Edit:` (a member glyph's resolved span, or the whole file for a file path) for a reference to the deleted symbol's identifier, and reports a match.
A reference is the identifier as a whole word, qualified as Go would spell it from that location: package-qualified (`pkg.Name`) from another package, bare or as a selector (`.Name`) within the same package.
It runs in the same validation entry points as D1 and is scoped the same way at dispatch.
A Delete target that is a file path, and a reference from code no card edits, are not checked.
The finding names the deleting card, the later card, the target and the file and line of the reference, and tells the writer to move the delete to a card after the later one.

Rationale: quarry has no reference API, so a whole-word match inside the later card's resolved spans is the narrowest test that catches both sightings (`pair-lifecycle`, #401) without a new quarry feature.

Bound: the match is textual, so it can flag an unrelated symbol of the same name inside a later card's span; the fix it names, moving the delete later, is always available and harmless, which is what makes a blocking false positive acceptable.
It never fires on code outside a later card's `Edit:` targets.

### D3 — A Delete target already gone at dispatch is not blocking

Decision: in `begin-batch`'s dispatch validation (`planglyph.ValidateDispatch` and the checks it runs), a **Delete:** target of a card not yet begun that no longer exists (path missing, or glyph not found) becomes an informational advisory, returned in `BeginResult.Advisories` and so in the batch's begin output, naming the target as already deleted.
The Plan-Gate and `lyx webster validate` before the run starts keep refusing a missing Delete target.

Rationale: an absent Delete target is the state the card wants (`webster-delete-already-gone`); at Plan time the same absence is a typo in the plan, so the relaxation applies only once the run has started changing the tree.

Bound: only **Delete:** targets, only in the dispatch validation; a missing `Edit:`, `Uses:` or Rename-old target still blocks.

### D4 — A delete-not-done caused by a later card's reference names that cause

Decision: when `record-batch`'s done-check reports `delete-not-done` and D2's check finds the target still referenced from a later card's `Edit:` code, the batch-failed reason names that later card and the reference, and the failure's way forward is: move the delete to a card after that one, run `lyx webster rebaseline --card NN` naming the edited cards, then `lyx webster recover-batch NN`.
`recover-batch` runs the same check over its batch's cards before spawning a recovery strand, and refuses with that same way forward while the check still fires, so it never names itself as the way forward from a state it cannot change.
Every other batch failure keeps `recover-batch` as its way forward.

Rationale: in #401 the refusal named `recover-batch`, whose recovery strand faced the same impossible delete, and the cause (card order) was never named.
With D1 and D2 such a plan should not reach Webster, but a plan edited and rebaselined mid-run can, and the refusal must not loop.

Bound: `recover-batch` refuses before spawning only while D2's check fires for the batch's own cards against the current plan and tree.

### D5 — begin-batch archives a report that has no begin-batch record

Decision: when `begin-batch` finds a report on disk for a batch that state.json has no record for, it archives the report through the existing report archive (`archiveStaleReport`'s timestamped rename, the one `record-batch`'s unattributable path uses) and proceeds; the archived path is returned in the begin result.
When state.json does record the batch, the existing refusal stays, and its message now states the record it saw (begun and not terminal, or terminal with its status) beside the way forward.

Rationale: #402's three refusals came from `skills-in-one-turn`'s recovery, which archived state.json by hand after `webster reset --to start` and left the reports from the abandoned batches under `_lyx/webster/reports/`; each redone batch then bounced begin-batch → record-batch → begin-batch.
A report with no begin record cannot be attributed by `record-batch` either, which already archives it, so begin-batch doing the same archive is the one step the loop was spending two refusals on.

Bound: only when no record for that batch number exists; a recorded batch's report is never archived by begin-batch.

### D6 — rebaseline --card accepts an edit to a failed or dead batch's card

Decision: `Rebaseline` accepts a changed card hash for a begun batch when the batch's record is terminal with status `failed` or `dead` and `--card` names that card; it restamps that card's hash in the batch record and keeps the rest of the record (StartSHA, warnings, transcripts).
The batch's recorded card-ID set must still match the plan.
For a batch that is `done`, or begun and not terminal, the refusal stays and says why `--card` cannot accept it: a done batch's work has landed, and a non-terminal batch may have a fork in flight; for the non-terminal case it names `lyx webster record-batch NN` or `lyx webster recover-batch NN` first.

Rationale: #403's refusal offered only `reset --to start` plus `run --fresh` for a one-card fix; a failed or dead batch has no fork running and is re-driven by `recover-batch`, which renders from the current plan, so the edited card is what the recovery executes.

Bound: only cards named with `--card`, only in batches terminal `failed` or `dead`, never a change to a batch's card-ID set and never `00-overview.md`.

### D7 — webster reset moves the remote task branch with the local one

Decision: `lyx webster reset`, after fabric resets the pair's warp checkout, also updates the remote task branch to the reset commit, through a new lease-guarded remote update in `internal/fabricengine/destroy.go`, run through `gitexec`.
Before any mutation, reset fetches the remote task branch and refuses when the remote tip holds a commit not reachable from the pre-reset local HEAD; the refusal names the foreign commits and the way forward (integrate them, then re-run the reset).
The remote update is skipped when the remote tip already equals the reset commit, when the remote has no such branch, and under `SkipPush`.
The parent branch is never updated: the existing foreign-branch refusal keeps a reset off it, and the remote update re-checks it.

Rationale: #404's rejected push was not a race on the parent; Publish pushes only the warp task branch, and fabric's sync had pushed the batches `skills-in-one-turn` later abandoned with `webster reset --to start`, so the remote task branch held commits the rewound local branch lacked.
Every later push was rejected (coalesce only warned), and Publish halted.
A force update is denied to agents, so lyx performs it itself (`PATTERN-no-denied-recovery`), guarded so it can discard only commits the run itself made and pushed.

Bound: the remote update rewrites only the pair's own task branch, only to the commit the reset moved the local branch to, only when every commit it drops is reachable from the pre-reset local HEAD, and with a lease on the remote tip read just before.

### D8 — Publish's push-rejected halt states the cause and the resume

Decision: Publish keeps escalating on `gitrepo.ErrPushRejected` with no retry.
Its stuck reason names that the merge-in against the parent already ran, the remote task branch's tip and the number of commits it holds that the local branch lacks, and the way forward: merge `origin/<task-branch>` into the task branch in the task worktree, then resume the run with the existing loom resume verb.

Rationale: Publish merges the parent in before pushing, so a refetch and retry cannot change the outcome; the only cause left is the remote task branch itself diverging, which after D7 means commits pushed from outside the run, and integrating them is a decision for whoever pushed them.

### D9 — The background-shell note states the outcome

Decision: the behavior stays: once a non-awaited background shell has been outstanding for `background_shell_wait_min`, shuttle counts Master's turn end and lyx does not stop the shell.
The `webster-background-shell` friction note now states, per shell, that the wait stopped waiting after the bound and counted the turn end, that lyx did not stop the shell, what ends it (the end of Master's session, as the plan establishes it), and the run's outcome after that turn end (done, stuck with its reason, or the error).
The same wording reaches the `RunResult.Warnings` entry.

Rationale: waiting longer can hang the run on a shell that never exits, killing a provider's background shell needs a provider mechanism shuttle does not have (a new feature), and escalating would turn finished runs into stuck ones over a shell that may be harmless; #397's actual defect is that the note gave the operator no outcome.

### D10 — begin-batch drops its per-batch model assertion

Decision: `begin-batch` no longer asserts or injects a model: the model-switch injection, `State.AssertedModel`, `BeginResult.AssertedModel`, `BeginDeps.Injector` and its `Roles`/`Engine` use for that purpose are removed, together with the refusal-spec row "model switch injection failed" and the package doc's "only model-injection site" text.
A state.json written before this change, still carrying `assertedModel`, loads unchanged.

Rationale: the fork inherits Merriam's model, so `begin-batch` has no model to check (operator, `webster-batch-validation-speed`).

### Added after Discussion (operator, 2026-10-06): Both test prunes land before implementation

Decision: Webster for this task starts only after test-suite-prune-core and test-suite-prune-rest have both landed on main; the plan writer reads both landed states, since prune-rest rewrites the landingshed, fabric and planparser tests this task extends.

Rationale: Operator decision 2026-10-06: the two prunes rewrite the tests this task's regression tests build on, so implementing first would conflict with them.

## Constraints

- `PATTERN-refusal-way-forward`: every changed or new refusal (D4, D5, D6, D7, D8) gets its `contracts/specs/refusal-spec.md` row in the same commit and a test that reaches it.
- `PATTERN-no-denied-recovery`: no way forward names `git push --force`, `git reset --hard` or `rm -rf`; D7 is lyx performing the force update itself.
- `PATTERN-fabric-git`, `PATTERN-fabric-destruction-chokepoint`, `PATTERN-gitrepo-client-boundary`, `PATTERN-gitexec-checked-call`: D7's remote update lives in `internal/fabricengine/destroy.go`, checks containment, ownership, dirtiness and force in that order, and runs its remote git through `gitexec`; `websterengine` stays fabric-blind.
- `PATTERN-mutation-record`: D7's remote update appends to the reset's `*Mutations` record.
- `PATTERN-sole-parsers`: D1 lives in `internal/planparser`; D2 reads cards only through the parsed plan.
- New plan checks get rows in `contracts/specs/loom-plan-spec.md`'s "Validation checks" list and in the owning package's `doc.go` check list.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-hubforge-fixtures`: each regression test sits in the lowest tier its substrate needs; D7's test builds its hub through `hubforge`.
- Board instruction: a regression test per fix drives the real CLI wiring (the `lyx` cobra command), not only fakes.
- Board instruction: no new verbs, config keys or features.
- The task depends on `test-suite-prune-core`, which reshapes `internal/websterengine`'s tests; new tests fit the pruned shape.
- `PATTERN-comment-line-breaks` and semantic line breaks in every changed `.md`.

## Auto-mode assumptions

- #402's reports came from the hand recovery of `skills-in-one-turn` (inferred from the weft history: `report-archived` records for batches 01, 02 and 04 right after the 09:48 re-seed); D5 holds whatever wrote them.
- #404's diverged remote came from `webster reset --to start` rewinding a pushed task branch (inferred from Publish pushing the warp task branch only and fabric pushing it mid-run); D8 holds whatever the cause.
- The field rule's inclusion of **Delete:** targets (D1) is accepted although a card that only reads a symbol a later card deletes is executable in card order; it goes away with `webster-delete-ordering`'s sequencing drop.
- The plan establishes what ends Master's session after Webster finishes, and D9's note states that truthfully.

## Open risks

- D2's textual match can flag a same-named, unrelated symbol; the cost is one plan-writer reorder.
- D2 does not see a reference from code no later card edits, or a symbol defined in a deleted file; those still surface at the batch's verify or the Burler.
- D1 can refuse plans that today run only because `SequenceBatches` reorders them, so Plan-Write may bounce more until `webster-delete-ordering` updates the plan spec's prose and template.
- D7 adds a remote-rewriting step to fabric; a wrong ancestry check would discard someone else's pushed commits, which is why the refusal-before-mutation and the lease are part of the decision.

## Acceptance criteria

- A plan where card 2's `Uses:` names a target card 4 edits (the `skills-in-one-turn` shape) fails the Plan-Gate with D1's finding naming cards 2 and 4; the same plan with the cards swapped passes.
- A plan where card N deletes a symbol that card N+1's `Edit:` code still references fails with D2's finding naming both cards and the reference; with the deleting card numbered after the editing card (the `pair-lifecycle` order) it passes.
- A cyclic plan of the `skills-in-one-turn` shape never reaches Webster.
- A Delete target removed by an earlier batch does not stop the later card's `begin-batch`; its output carries the already-deleted advisory; the same missing target in a plan not yet run still fails `lyx webster validate`.
- A `record-batch` failing on `delete-not-done` because a later card references the target names that card and the plan-edit way forward, not `recover-batch`; `recover-batch` on that batch refuses before spawning with the same way forward.
- `begin-batch` on a batch with a report on disk and no begin-batch record archives the report and succeeds; with a record present it refuses and names the record's state.
- `rebaseline --card NN` on a card of a failed batch succeeds and `recover-batch` then renders the edited card; on a done batch, and on a begun non-terminal batch, it refuses with D6's reason.
- `webster reset --to start` after pushed batches leaves the remote task branch at the reset commit, and a following push succeeds; with a foreign commit on the remote task branch, reset refuses before any mutation.
- Publish's push-rejected stuck reason names the remote tip, the remote-only commit count and the merge-then-resume way forward.
- The `webster-background-shell` note and warning state that the turn end was counted after the bound, that the shell was not stopped, what ends it, and the run's outcome.
- `begin-batch` injects nothing into Master's pane, and a state.json with `assertedModel` still loads.
- Each item above is driven through the `lyx` CLI command at least once.
- Every changed refusal has its `refusal-spec.md` row; the new checks have their `loom-plan-spec.md` rows.

## Notes for the plan writer

- `deriveEdges` and `refsIntersect` in `internal/websterengine/sequence.go` define the rule D1 must match; the check's own copy lives in `internal/planparser`.
- `planglyph.ValidateFormat`, `Validate` and `ValidateDispatch` are the three entry points; `internal/loomshed/gates.go` wires the gates.
- `internal/websterengine/batchfail.go` (`BatchFailedError`) renders the batch-failed way forward D4 changes; `recoverbatch.go` holds `recover-batch`'s spawn path.
- `internal/websterengine/beginbatch.go` holds the report refusal (D5) and the model injection (D10); `rebaseline.go` holds D6.
- `internal/webstercli/reset.go` calls `fabricengine.Fabric.ResetPairWarp`; `fabricengine.Fabric.PushBranch` and `coalesce.go` show the existing push path.
- `internal/landingshed/publish.go` Step 5 holds the push-rejected stuck reason (D8).
- `internal/websterengine/triage.go` writes the background-shell note and warning (D9).
