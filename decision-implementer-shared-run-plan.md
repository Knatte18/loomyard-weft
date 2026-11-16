## Goal

Move Webster's batch machinery and run-plan state into a package both implementers import

Takes: Webster's batch machinery as it stands in internal/websterengine: report.go, digest.go, fingerprint.go, the plan-conflict refusal of beginbatch.go and conflictrecord.go, postBatchChecks in recordbatch.go, suspect.go, cardgate.go, awaitbatch.go and strand.go, verifygatereport.go and verifyparse.go, and the run-plan fields of state.go's State and BatchState
Delivers: a new package internal/runplan holding the run-plan state (plan fingerprint, plan file hashes, overview frame hash, recorded partition, batch records) that websterengine.State embeds beside its Master fields, with every Webster-only batch-record field kept in websterengine or optional; the batch report contract and parser with an optional journal field; the plan fingerprint and plan-conflict refusal; the post-batch checks; the suspect-path checks; the failure digest; card gate rendering; strand await; and the verify-failure record. Each shared piece takes the implementer's run, reports, prompts and scratch directories and its state-mutation lease from a geometry the caller passes, and each shared refusal takes its way-forward text from the caller, so Webster passes its own and its behaviour and existing tests stay unchanged
Exports: package internal/runplan: the run-plan state type and batch-record type both implementers' state files embed, the report type with Journal and its parser, the post-batch check entry point taking a recorded partition mapped onto the current plan, the suspect check, the digest, the card gate renderer, the strand await, the verify-failure record, the plan-conflict error and record, and the implementer geometry type; websterengine keeps aliases for the names it moved so its importers compile unchanged

## Scope

The part changes these packages:

- internal/runplan (about 1600 changed lines)
- internal/websterengine (about 900 changed lines)
- internal/webstercli (about 200 changed lines)
- internal/loomshed (about 80 changed lines)
- internal/loomcli (about 80 changed lines)
- internal/shedadapters (about 60 changed lines)
- internal/failuretrace (about 30 changed lines)
- internal/hubgeom (about 20 changed lines)
- internal/standalonegeom (about 20 changed lines)
- tools/tokencount (about 20 changed lines)

## Decisions

### D11 — Webster's batch machinery is shared, not copied

Weaver reuses Webster's report contract and parser, plan fingerprint and plan-conflict refusal, post-batch checks (done checks, handle binding, scope guard, renames rewriting pending cards), suspect-path checks, failure digest, card gate rendering, strand await and verify-failure record.
The state types the shared pieces read move with them into a package both implementers import: a run-plan state holding the plan fingerprint, plan file hashes, overview frame hash, the recorded partition and the batch records.
Both implementers' state files embed it; Webster's `State` keeps its Master fields beside it, and a batch-record field only Webster fills stays optional and empty under Weaver.
Every told directory of `Geometry` that names Webster's run, reports, prompts or scratch directory names the active implementer's under Weaver, so Weaver's state lock and state-mutation lease sit under `.lym/weaver/`.
The batch list Weaver hands the shared checks is its recorded partition mapped onto the current plan, the same mapping Webster's verbs use, so the later-reference check sees the pending cards and only the run's last batch reads as last.
Under Weaver, every shared refusal names Weaver's way forward: a plan-partition refusal names D7's, never Webster's rebaseline or restore-plan verbs.
Webster's behaviour stays as it is.
Weaver's report adds `journal` to the shared contract; Webster's forks may leave it empty.
Fork transcript audits stay Webster's alone.
Rationale: PATTERN-shed-driver's shared-machinery rule; two copies of the post-batch checks would drift.

### Added after Discussion (arbiter, 2026-11-16): One producer runs every batch (answers F11, Q17)

Decision: The record's reading stands: one Weaver row whose producer loops over the batches in Go, each batch's stop returned as the row's Stuck so it reaches shed's stop chain (rule table, tackler, arbiter, parent), meets the operator's 2026-11-15 decision that each batch is a producer. The writer states this in D2 as settled, drops the first Open risks line and marks Q17 answered by this decision.

Rationale: The operator's decision names its purpose: the Shed driver's tackler and arbiter handle a stuck batch as for any recipe, which D2's per-batch Stuck returns give. The board's own 'Weaver as a Shed run' idea lists one batch producer looping over the batch index, as the Burler round loops over rounds, as a way to meet it; the 2026-10-17 operator view puts the batchifier inside Weaver rather than in rows of its own; the implementer seam pins one row with fixed edges, and recipes are fixed files.

### Added after Discussion (arbiter, 2026-11-16): The journal is pushed with the run at each shed transition (answers F10, Q16)

Decision: Weaver keeps the implementer contract: it never pushes, and no push is added inside the producer. Its per-batch commits of _lym/weaver/ and the plan directory (D9), the journal included, and the batches' code commits are pushed by the run's own push at the row's end, the push every shed transition makes, Done or Stuck alike. The writer states this in D9 after 'Weaver never pushes', drops the second Open risks line and marks Q16 answered by this decision. push-before-act-takeover's scope is not changed.

Rationale: The board says the journal is pushed with the run, that is by the run's push alongside the code it describes, and outlives a crash or a move to another machine. A crash keeps the per-batch commits locally, and a move to another machine goes through a stop: the adopt redesign's run FSM leaves every stopped run in a pushed state another machine goes on from, and a Weaver stop is a shed transition that pushes both sides. A push from Weaver would break implementer-spec's never-pushes line, and a push inside a producer would widen push-before-act-takeover, another task's scope.

### Added after Discussion (arbiter, 2026-11-16): The row's gates entries need not run over a verify fixer's commit (answers Q18)

Decision: D10's bound stands: a verify fixer runs through gatefixer with no shuttle gates, and the row returns Done when the plan-level verify passes after the last fixer, with no gated strand added over a fixer's commit and no PATTERN-gate-slot change. The Open risks question and support-log Q18 are answered by this decision; the plan writer builds D10 as written and adds no extra spawn for the row's gates.

Rationale: implementer-spec.md pins the implementer's Done as 'the plan is built and the plan-level verify passed', which D1 keeps and D10's Rationale cites; the row's gates entries are not part of that contract. PATTERN-gate-slot, as the record's Constraints line states it, runs a cold verify fixer only through gatefixer, so the review's option (b) would need a PATTERN change the record chose against in round 3. The one gates entry the shipped tapestry recipe gives the implementer row, drift, is repeated downstream by the Drift-Verify row after Parent-Align as the guarantee (contracts/recipes/tapestry-recipe.yaml), so a drift the fixer's commit causes is still caught before review. An extra gated strand over each fixer commit would add a spawn and its own crash and retry rules to a path the review just closed.

## Constraints

- PATTERN-shed-recipe-registry: the `Implementer` entry's closed table is the one exception to the registry map; Weaver joins that table and adds no registry entry.
- PATTERN-shuttle-provider-seam: rung specs reach a spawn only as `ModelChoice` built by `NewModelChoice`; strands stop only through shuttle's stop verb; a negative finish consults the output files first.
- PATTERN-reed-panes: every batch and fixer strand is a reed session started through shuttle, never headless.
- PATTERN-fabric-boundary: the per-batch commit of `_lym/weaver/` and the plan directory goes through fabric, as Webster's run commit does.
- PATTERN-config-layers: `weaver.yaml` is a new module loaded through `configengine` and registered in `configreg`; its template ships the defaults.
- PATTERN-lym-files: `_lym/weaver/`, `.lym/weaver/` and the shared plan-conflict path are named only by `lymdirs`; locks through `lock`.
- PATTERN-tolerant-records: Weaver's state file never refuses an unknown key.
- PATTERN-gate-slot: the plan-level verify runs in held gate slots through `verifytree.Verify`; its cold fixer runs only through `gatefixer`, one per failed evaluation, none for a runner crash, and Go's re-run judges it (D10).
- PATTERN-stencil-ownership: Weaver's prompts are stencils read at call time through `stencilstore`, rendering the parent and edit directives.
- PATTERN-refusal-way-forward: every Weaver refusal and Stuck reason names its way forward, and a test reaches it.
- A Codex spec may stand on a rung only as far as `codex-same-as-claude` has given Codex sessions what Claude sessions in the same role get; the default ladder is Claude-only.

## Auto-mode assumptions

- The loop over batches lives in the row's producer (D2).
- Batches run sequentially in the task worktree (D4).
- The default ladder is `sonnet[medium]`, `opus[medium]`, `opus[high]` (D5).
- A machinery loss retries once on the same rung (D6).
- A re-entered ladder stop gets one attempt on the last rung; a re-entered `NEEDS_PARENT` stop resumes at the rung that asked (D7).
- The journal is a report field Go appends (D8).
- The verify reuses the `webster` gate key by default (D10).

## Open risks

## Acceptance criteria

- Webster's existing tests pass unchanged after the shared machinery and its state types move (D11).
