## Interview

### Turn 1 — autonomous exploration

No operator answers this run; every question was settled by the writer.
Read: the board entry, `plan-drift-refusal-route`, `glyph-use-audit`, `internal/planglyph/donecheck.go`, `postBatchChecks` in `internal/websterengine/recordbatch.go`, `internal/websterengine/cardgate.go`, `internal/batcher/doc.go`, `planparser.CardTargetDirs` and `RefScope`, the plan spec's card-type table and Rename and Move section, and quarry's `Repo` API.
Findings that shaped the design:

- `DoneChecks` resolves every card of a batch against the worktree at batch end, and quarry's `Resolve` has no revision argument.
- The plan spec already says a card that renames or deletes what an earlier card targets never shares that card's batch; the cost batchifier's feasibility test checks only size and budget.
- `CardTargetDirs` treats a `Move` ref as a live target, and `gateScopeOf` checks `.go` presence as the tree stands at begin-batch.
- `twb-d0-ga` and `twb-d0-gc` carry this entry's body; they are parallel bench arms of the same task, not separate work.

### Turn 2 — lens review

Six lens forks reviewed the record written by fork0; every fork returned a report.

- premises BLOCKING: the spec sentence D1 cited as present in `contracts/specs/loom-plan-spec.md` is absent at HEAD (`contracts/specs/loom-plan-spec.md:130` runs straight into `ImpactSummary`), present only in the hub copy `_board/_lyx/specs/loom/loom-plan-spec.md:133` — fixed as proposed: D1 cites the hub copy and states the HEAD source lacks it, D4 adds the sentence, Open risks names a parent change that may already touch the batcher.
- premises MEDIUM + pattern-fit BLOCKING (merged): "this repo's config layer" names no writable layer; the repo default is `<Board>/_lyx/config/webster.yaml` (`internal/configengine/doc.go`), and the worktree holds no `webster.yaml` — fixed as proposed: D3 says the key enters the board default through the template and hub reconcile, and the operator or orch sets `./cmd/lyx` after deploy; Scope Out names it.
- pattern-fit BLOCKING: the key has no `ConfigSpec` category (`internal/websterengine/config.go:55-63`) — fixed as proposed: D3 classifies it `CategoryGuard`, the category for a key that hollows a gate (`internal/configengine/layers.go`).
- pattern-fit MEDIUM: an older binary reading the new key is not addressed — fixed differently: proposed was a binary-compatibility analysis; instead D3 and Constraints route the key through the template and `PATTERN-binary-change-reconcile`'s hub reconcile and set the value only after deploy, so no older binary meets it.
- pattern-fit LOW: `PATTERN-leaf-packages` does not bind `internal/batcher` — fixed as proposed: line dropped.
- pattern-fit LOW: D4 should name the spec source file — fixed as proposed.
- pattern-fit LOW (D2 has no conflict) — no change; it reports a confirmation, not a defect.
- premises MEDIUM: D2 attributed the disk check to `CardTargetDirs`; `gateScopeOf` does it through `holdsGoFile` — fixed as proposed.
- premises LOW: a `Create` handle names a file unit (`handle-malformed`), so the second hit rule's rationale should rest on package-unit glyphs — fixed as proposed.
- premises LOW: package self-glyph `Rename` pair admissibility unconfirmed — fixed as proposed: no record change, the lens itself proposed confirming it at Plan time.
- scenarios MEDIUM: no bound on malformed `batch_gate_packages` values — fixed as proposed: D3 defines the entry grammar, a load-time refusal with way forward, and the bound; acceptance criterion added.
- scenarios MEDIUM + goal-scope MEDIUM (merged): a `cmd/lyx` failure the batch did not cause — fixed as proposed: D3 states the failure is the batch's under the self-fix and recovery rules, Open risks names a red base, which the plan-level verify gate stops today too.
- scenarios MEDIUM: D2 said "at begin-batch" though recover-batch renders from the current tree (`recoverbatch.go:281`) — fixed as proposed: "when the gate is rendered", idempotence stated, acceptance criterion added.
- scenarios MEDIUM: acceptance gaps (create-then-delete, card gate, identity, all-barred, package rename) — fixed as proposed: criteria added.
- scenarios LOW: behaviour on a failed webster config load — fixed differently: proposed was a fallback-or-refuse decision; the existing degrading load already decides it (`PATTERN-config-strictness`), so D3 only adds the per-value refusal.
- scenarios LOW: redundant untagged run when the batch's scope holds a key package — fixed as proposed.
- scenarios LOW: re-partition after rebaseline, reset or `run --fresh` — fixed as proposed: D1 says every batchifier call applies the barrier.
- goal-scope MEDIUM: no cost figure for `./cmd/lyx` per batch — fixed differently: proposed was stating a duration; no measured figure exists and this session runs no raw go test, so Open risks says so and names where the run time is recorded and the operator's off switch.
- goal-scope LOW: D2 credited with the incident D1 fixes (card 12 is file `Rename` pairs) — fixed as proposed: D2's rationale names its own case, D1 walks the incident.
- goal-scope LOW: link scope items 1 and 2 to D1 — fixed as proposed: Goal sentence added.
- goal-scope LOW: `Custom` absent from the removed set — fixed as proposed: D1 names it an accepted gap; acceptance criterion added.
- board-fit MEDIUM: D3 overlaps `dotnet-scoped-verify` — fixed as proposed: D3 says Go package arguments, Scope Out names `dotnet-scoped-verify`.
- board-fit LOW: `weaver` reuses partition and recovery prompt — fixed as proposed: one clause in Constraints.
- board-fit LOW: disjointness from `delete-before-reference` — fixed as proposed: clause in Scope Out.
- leanness LOW: Open risks 1 and 3 restate D1 and D2 — fixed as proposed: both dropped.
- leanness LOW: D2 rationale narrates code — fixed differently: proposed was cutting the sentence; it now carries the premise the D2 fix rests on (render-time presence, `Move` live), corrected per premises, so it stays shortened.
- leanness LOW: D4 restates repo doc rules — fixed as proposed: D4 keeps only the spec and template requirements.
- leanness LOW: test-budget constraint restates CLAUDE.md — fixed as proposed: cut.
- leanness LOW: Notes repeat D2 — fixed as proposed: only the `cost.go` pointer stays.
- leanness LOW: Out duplicates Rejected alternatives — no change; the lens itself proposed none.

## Rejected alternatives

- **Resolve each card's glyphs at its own commit.** Quarry's `Resolve` reads the working tree only; this needs a scratch checkout per card or a quarry change, for a case the barrier removes.
- **Follow renames in `DoneChecks` and `BindHandles`.** A `Move` destination lives in `Intent` prose, so the mapping is not machine-readable for moves; file-by-file rename pairs make a package rename a prefix-mapping problem with its own failure modes.
- **Plan-time pin detection in `validate-plan`.** A by-name pin can be grepped for heuristically, but a repo-wide scan such as the tier-purity sleep check cannot be predicted from a card's targets.
- **Run `./cmd/lyx` hard-coded in every batch gate.** lyx drives other repos; which package holds the scans is per repo.
- **`lyx gate test` skipping a missing package.** It would also hide a typo in a hand-written gate.
- **A begin-batch assertion of the barrier.** It would refuse in-flight runs whose partition an older binary recorded.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): how the done-check stops failing on a later in-batch rename. Pick: the batcher barrier (D1). Rationale: it enforces a rule the plan spec already states, keeps `DoneChecks` a single working-tree resolve, and covers the gate case too.
- Q2 (auto-pick): does `Move` count as a removal. Pick: yes, for the barrier and the gate scope. Rationale: a `Move` removes its target from its source as a `Rename` does.
- Q3 (auto-pick): how a `cmd/lyx` scan break surfaces at the batch that causes it. Pick: `webster.yaml` `batch_gate_packages`, set to `./cmd/lyx` here, run untagged (D3). Rationale: catches scans and pins alike; cost is one package run per batch, removable by config.
- Q4 (auto-pick): re-assert the barrier at begin-batch. Pick: no. Rationale: the batcher is the one place the batch is derived (`PATTERN-batcher-registry`), and recorded partitions never regroup mid-run.
- Q6 (auto-pick): which category and layer hold `batch_gate_packages`. Pick: `CategoryGuard`, empty template, the board's repo default set to `./cmd/lyx` by the operator or orch after deploy. Rationale: emptying it drops a gate step, and the task may not write `_lyx/config/` of the run that spawned it.
- Q5 (auto-pick): add a verb to accept a batch by hand. Pick: out of scope. Rationale: the entry's scope-to-settle list does not name it, and the barrier removes the case that needed it.
