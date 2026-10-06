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
The finding is attributed to the consumer card, the one whose `Uses:` matches, and names both cards and the shared entry; its text tells the writer to move the work or reorder the cards so the producer comes first.
At dispatch `planglyph.ValidateDispatch` drops a pure finding by the card it is attributed to, so a pair is reported only when its consumer card is unbegun, one of a batch with no record in state.json (the existing dispatch scoping); every later check below uses the phrase "unbegun cards" for that scope.
It compares cards by number whatever batch they sit in: under the identity batcher each card is its own batch, so this is `deriveEdges`' cross-batch rule, and under a grouping batchifier a same-batch pair is also refused, accepted because a valid plan is executable in card-number order.

Rationale: a valid plan is executable strictly in card-number order (operator rule, 2026-10-06).
`deriveEdges`' other edge kind, two cards sharing a target, always points from the lower number to the higher, so once no `Uses` edge points backward every derived edge points forward.
A cycle needs a backward edge, so no separate cycle check exists, and `SequenceBatches` then returns the declared order unchanged (its documented no-op property): Webster already runs gated plans in card order without its sequencing being touched in this task.
That holds when every batch is a contiguous run of card numbers, as the identity batcher's batches are; the claim goes no further.
Mirroring `deriveEdges` exactly, not a narrower rule, is what guarantees that: any edge the gate let through backward would still be reordered by Webster.

Bound: this refuses plans that pass today; the bound is that it refuses only a `Uses`/later-target match that would already make Webster reorder the cards.

### D2 — Plan-Gate refuses a delete that runs before a later card removes a reference to it

Decision: a new blocking plan check, in `internal/planglyph` because it reads the tree: for every **Delete:** target of card N that is a member or unit glyph, it searches the code each card numbered above N names under `Edit:` (a member glyph's resolved span, or the whole file for a file path) for a reference to the deleted symbol's identifier, and reports a match.
A reference is the identifier as a whole word, qualified as Go would spell it from that location: package-qualified (`pkg.Name`) from another package, bare or as a selector (`.Name`) within the same package.
It runs in the same validation entry points as D1 and is scoped the same way at dispatch (unbegun cards on both sides).
A Delete target that is a file path, and a reference from code no card edits, are not checked.
The finding names the deleting card, the later card, the target and the file and line of the reference, and tells the writer to move the delete to a card after the later one.

Rationale: quarry has no reference API, so a whole-word match inside the later card's resolved spans is the narrowest test that catches both sightings (`pair-lifecycle`, #401) without a new quarry feature.

Bound: the match is textual, so it can flag an unrelated symbol of the same name inside a later card's span; the fix it names, moving the delete later, is always available and harmless, which is what makes a blocking false positive acceptable.
It never fires on code outside a later card's `Edit:` targets.

### D3 — A Delete target already gone at dispatch is not blocking

Decision: in `begin-batch`'s dispatch validation (`planglyph.ValidateDispatch` and the checks it runs), a **Delete:** target of an unbegun card that no longer exists (path missing, or glyph not found) becomes an informational advisory, returned in `BeginResult.Advisories` and so in the batch's begin output, naming the target as already deleted.
The Plan-Gate and `lyx webster validate` before the run starts keep refusing a missing Delete target.

Rationale: an absent Delete target is the state the card wants (`webster-delete-already-gone`); at Plan time the same absence is a typo in the plan, so the relaxation applies only once the run has started changing the tree.

Bound: only **Delete:** targets, only in the dispatch validation; a missing `Edit:`, `Uses:` or Rename-old target still blocks.
What it lets through: a Delete target missing for any reason, including one a card removed by editing the wrong file or one the tree never held, passes with the advisory alone.
What bounds that: the Plan-Gate and `lyx webster validate` saw the target exist before the run started, so only a target the run removed, or one a mid-run plan edit added, can reach this state, and the advisory names the target and the card.
`record-batch`'s `delete-not-done` check does not bound it, since it reads a target that is not found as the wanted state.
The Rework-Plan-Gate runs before dispatch and keeps refusing a missing Delete target, so a rework plan that still lists a target an earlier batch removed drops it from the card.

### D4 — A delete-not-done caused by a later card's reference names that cause

Decision: when `record-batch`'s done-check reports `delete-not-done` and D2's check finds the target still referenced from a later card's `Edit:` code, the batch-failed reason names that later card and the reference, and the failure's way forward is: move the delete to a card after that one, run `lyx webster rebaseline --card NN` naming the edited cards, then `lyx webster recover-batch NN`.
The check runs over the batch's own cards as the deleting side, although they are begun, against later unbegun cards, the scope D1 names.
`recover-batch` runs the same check over its batch's cards before spawning a recovery strand, and refuses with that same way forward while the check still fires, so it never names itself as the way forward from a state it cannot change.
That check runs after `recover-batch`'s refusal of a failed batch carrying uncheckable paths, which keeps its own way forward (`run --fresh`), so the D4 refusal, whose way forward ends in `recover-batch NN`, is reached only by a batch that recover-batch can accept.
When the failing batch's record also lists uncheckable paths, the batch-failed reason names the `run --fresh` steps that `recover-batch`'s own refusal names instead of the plan-edit way forward, since `recover-batch` would refuse that batch.
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

Decision: `Rebaseline` accepts a changed card hash for a begun batch when the batch's record is terminal with status `failed`, `dead` or `stuck` and `--card` names that card; it restamps that card's hash in the batch record and keeps the rest of the record (StartSHA, warnings, transcripts).
The batch's recorded card-ID set must still match the plan.
For a batch that is `done`, or begun and not terminal, the refusal stays and says why `--card` cannot accept it: a done batch's work has landed, and a non-terminal batch may have a fork in flight; for the non-terminal case it names `lyx webster record-batch NN` or `lyx webster recover-batch NN` first.

Rationale: #403's refusal offered only `reset --to start` plus `run --fresh` for a one-card fix; a failed, dead or stuck batch is terminal and is re-driven by `recover-batch`, which stops any live prior strand (a dead batch's strand is kept alive by design) and renders from the current plan, so the edited card is what the recovery executes.
Where `recover-batch` itself refuses the batch (a failed batch with uncheckable suspect paths, which it sends to `run --fresh`), the restamp cannot make the batch re-drivable, so `rebaseline --card` refuses that batch too and names the same `run --fresh` steps.

Bound: only cards named with `--card`, only in batches terminal `failed`, `dead` or `stuck`, never a failed batch whose record lists uncheckable paths, never a change to a batch's card-ID set and never `00-overview.md`.
What it lets through: for a dead batch the restamp lands while its strand, kept alive when classified dead, may still be working on the old card.
The restamp stops no strand; `recover-batch`, the next step the refusal names, stops that strand before it spawns and archives a late report from it, which ends the window.

### D7 — webster reset moves the remote task branch with the local one

Decision: `lyx webster reset` also updates the remote task branch to the reset commit, through a new lease-guarded remote update in `internal/fabricengine/destroy.go`, run through `gitexec`.
The remote update runs inside the pair-warp reset, in this order: the gate checks (containment, ownership, dirtiness), then a fetch of the remote task branch and an ancestry check, then the remote update, then the checkout rewrite.
The first refusal an operator meets is therefore the local, cheap one, and a refusal at any step before the remote update leaves the remote and the local branch unchanged.
The ancestry check refuses when the remote tip holds a commit not reachable from the local HEAD; the refusal names those commits.
That refusal also covers a remote that already diverged from a local branch an earlier reset rewound (#404's own state), where the remote-only commits are the run's own abandoned ones.
Lyx cannot tell whose commits they are in that state, because the local HEAD no longer holds them, so the refusal names the way forward for each case and the operator's judgment from the listed commits is the guard:
the commits are the run's own abandoned ones: `git merge --strategy ours origin/<task-branch>` in the task worktree, which takes no content and cannot conflict with the batches the run redid, then re-run the reset; the merge makes them reachable from HEAD and the re-run rewinds both, so the merge is only a hand-off and the re-run discards what it merged;
the commits may be someone else's work: do not re-run the reset, which would discard them, and the refusal leaves the rewind to the operator.
No plain merge is offered, since the re-run would discard what it merged.
The guard applies only to a merge the operator runs.
Its lease is on the exact remote tip the ancestry check was run against, so a remote tip that moved after the fetch makes the update refuse, with the same way forward, and nothing has changed yet.
A fetch or remote update that fails for another reason (network, authentication) refuses the same way, changing nothing, with the way forward of re-running the reset once the remote is reachable.
If the checkout rewrite itself then fails after the remote moved, the remote sits at an ancestor of the local HEAD, a later push fast-forwards, and a re-run of the reset converges; the failure message states that the remote task branch was already updated and names the re-run.
The remote update is skipped when the remote tip already equals the reset commit or is an ancestor of it, when the remote has no such branch, when the repository has no remote configured, and under `SkipPush`.
The parent branch is never updated: the existing foreign-branch refusal keeps a reset off it, and the remote update re-checks it.

Rationale: #404's rejected push was not a race on the parent; Publish pushes only the warp task branch, and fabric's sync had pushed the batches `skills-in-one-turn` later abandoned with `webster reset --to start`, so the remote task branch held commits the rewound local branch lacked.
Every later push was rejected (coalesce only warned), and Publish halted.
A force update is denied to agents, so lyx performs it itself (`PATTERN-no-denied-recovery`), guarded so it can discard only commits the local branch already holds; a remote holding commits the local branch lacks is refused until the operator, having judged them the run's own, has made them reachable.

Bound: the remote update rewrites only the pair's own task branch, only to the commit the reset moved the local branch to, only when every commit it drops is reachable from the local HEAD, and with a lease on the remote tip the ancestry check saw.
A remote-only commit becomes reachable only through the `ours` merge the operator runs after reading the listed commits; that judgment is the one thing the bound leaves to the operator.

### D8 — Publish's push-rejected halt states the cause and the resume

Decision: Publish keeps escalating on `gitrepo.ErrPushRejected` with no retry.
Its stuck reason names that the merge-in against the parent already ran, the remote task branch's tip and the number of commits it holds that the local branch lacks, and the way forward: merge `origin/<task-branch>` into the task branch in the task worktree, then resume the run with the existing loom resume verb.

Rationale: Publish merges the parent in before pushing, so a refetch and retry cannot change the outcome; the only cause left is the remote task branch itself diverging, which after D7 means commits pushed from outside the run, and integrating them is a decision for whoever pushed them.

### D9 — The background-shell note states the outcome

Decision: the behavior stays: once a non-awaited background shell has been outstanding for `background_shell_wait_min`, shuttle counts Master's turn end and lyx does not stop the shell.
The `webster-background-shell` friction note now states, per shell, that the wait stopped waiting after the bound and counted the turn end, that lyx did not stop the shell, what ends it (the end of Master's session, as the plan establishes it; when the plan establishes nothing, the note omits that clause and guesses none), and the run's outcome after that turn end (done, stuck with its reason, or the error).
The same wording reaches the `RunResult.Warnings` entry.

Rationale: waiting longer can hang the run on a shell that never exits, killing a provider's background shell needs a provider mechanism shuttle does not have (a new feature), and escalating would turn finished runs into stuck ones over a shell that may be harmless; #397's actual defect is that the note gave the operator no outcome.

### D10 — begin-batch drops its per-batch model assertion

Decision: `begin-batch` no longer asserts or injects a model: the model-switch injection, `State.AssertedModel` together with `Run`'s launch-time write of it, `BeginResult.AssertedModel`, `BeginDeps.Injector` and its `Roles`/`Engine` use for that purpose are removed, together with the refusal-spec row "model switch injection failed" and the package doc's "only model-injection site" text.
A state.json written before this change, still carrying `assertedModel`, loads unchanged.

Rationale: the fork inherits Merriam's model, and the assertion is dormant in the shipped flow: `Run` launches Master on the master role's resolved model and records it as `AssertedModel`, and `begin-batch`'s only target is that same model, so the comparison never diverges and nothing is ever injected (operator, `webster-batch-validation-speed`).

Bound: the removal lets nothing through in the shipped flow, since the comparison was between a recorded model and the model `Run` launched Master on and never read the live pane.
A Master pane that drifted by an operator's `/model` stays on that model for the rest of the run, as it did before; a hand-edited `assertedModel` no longer causes an injection.
Master's model is set once at spawn, and nothing else in `begin-batch` reads it.

### Added after Discussion (operator, 2026-10-06): Both test prunes land before implementation

Decision: Webster for this task starts only after test-suite-prune-core and test-suite-prune-rest have both landed on main; the plan writer reads both landed states, since prune-rest rewrites the landingshed, fabric and planparser tests this task extends.

Rationale: Operator decision 2026-10-06: the two prunes rewrite the tests this task's regression tests build on, so implementing first would conflict with them.

### Added after Discussion (parent, 2026-10-06): Plan now, re-check test targets after the prunes

Decision: Plan-Write writes the plan now against the current tree; the 'reads both landed states' clause of the prune decision applies before Webster, not before Plan-Write. Cards name production code as their main targets and test files by path rather than by test-function glyph where they can. Once both prunes land, the pair is synced with main and the plan's test targets are re-validated, re-planned only where a prune renamed or deleted them.

Rationale: The operator's intent was to use the waiting time for Discussion and Plan, with implementation held until the prunes land; blocking Plan-Write on the prunes defeats that.

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
- The plan establishes what ends Master's session after Webster finishes, and D9's note states that truthfully; if it establishes nothing, D9's note omits the clause.

## Open risks

- D2 does not see a reference from code no later card edits, or a symbol defined in a deleted file; those still surface at the batch's verify or the Burler.
- D1 can refuse plans that today run only because `SequenceBatches` reorders them, so Plan-Write may bounce more until `webster-delete-ordering` updates the plan spec's prose and template.
- D7 adds a remote-rewriting step to fabric; a wrong ancestry check would discard someone else's pushed commits, which is why the refusal-before-mutation and the lease are part of the decision.
  An operator who runs the `ours` merge on commits that are someone else's discards them on the re-run; the listed commits and the refusal's wording are the only guard.

## Acceptance criteria

- A plan where card 2's `Uses:` names a target card 4 edits (the `skills-in-one-turn` shape) fails the Plan-Gate with D1's finding naming cards 2 and 4; the same plan with the cards swapped passes.
- A plan where card N deletes a symbol that card N+1's `Edit:` code still references fails with D2's finding naming both cards and the reference; with the deleting card numbered after the editing card (the `pair-lifecycle` order) it passes.
- A cyclic plan of the `skills-in-one-turn` shape never reaches Webster.
- A Delete target removed by an earlier batch does not stop the later card's `begin-batch`; its output carries the already-deleted advisory; the same missing target in a plan not yet run still fails `lyx webster validate`.
- A `record-batch` failing on `delete-not-done` because a later card references the target names that card and the plan-edit way forward, not `recover-batch`; `recover-batch` on that batch refuses before spawning with the same way forward.
- `begin-batch` on a batch with a report on disk and no begin-batch record archives the report and succeeds; with a record present it refuses and names the record's state.
- `rebaseline --card NN` on a card of a failed, dead or stuck batch succeeds and `recover-batch` then renders the edited card; on a done batch, and on a begun non-terminal batch, it refuses with D6's reason; on a failed batch whose record lists uncheckable paths it refuses and names the `run --fresh` steps; on a dead batch whose strand is still live, a following `recover-batch` stops that strand before it spawns.
- A `delete-not-done` failure whose batch record also lists uncheckable paths names the `run --fresh` steps, not the plan-edit way forward.
- `webster reset --to start` after pushed batches leaves the remote task branch at the reset commit, and a following push succeeds; with a commit on the remote task branch that the local HEAD lacks (including the #404 state of a rewound local branch and a remote holding the run's abandoned commits), or a remote tip that moved after the fetch, reset refuses before any mutation and names the merge way forward, and after that merge a re-run converges; a local gate refusal (dirty tracked path) leaves the remote untouched and is met before the ancestry refusal; a fetch or remote update that fails on the network or authentication refuses and changes nothing; the refusal for commits that may be someone else's names no merge and says not to re-run the reset; after a failed checkout rewrite a re-run converges; a remote tip that is an ancestor of the reset commit, a branch absent on the remote, a repository with no remote and `SkipPush` each leave the remote alone.
- Publish's push-rejected stuck reason names the remote tip, the remote-only commit count and the merge-then-resume way forward.
- The `webster-background-shell` note and warning state that the turn end was counted after the bound, that the shell was not stopped, what ends it, and the run's outcome.
- `begin-batch` injects nothing into Master's pane, `run` records no `assertedModel`, and a state.json with `assertedModel` still loads.
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
