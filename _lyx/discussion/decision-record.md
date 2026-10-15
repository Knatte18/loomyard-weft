# Decision record — twb-d0-sf

## Goal

A scope correction made while a darn run is in flight reaches every later row of that run, and a darn pull request no longer always waits for the orch at PR-Gate.
Darn-Write writes a short decision record in loom's format before it writes code, `lyx loom decision add --by parent` works on a darn run once that record exists, the Darn review measures the diff against the board entry as amended by the record, and the darn recipe gains a PR-Align row.

## Scope

In:

- Darn-Write writes `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md` before any code change.
- The record and support-log format moves out of the Discussion stencil's Step 5 into a registered spec that both writers point at.
- `lyx loom decision add` works on a darn run, including while Darn-Write runs.
- An `Added after Discussion` entry, once committed, cannot be dropped from the record by a later edit; the check covers loom records too (D5).
- Darn-Write on every spawn, the Darn review (Darn-Bouncer, Darn-Burler-Review, Darn-Burler-Fix), darn's Describe and the alignment agent read the record.
- A `PR-Align` row in `contracts/recipes/darn-recipe.yaml` between Publish and PR-Gate.
- The orch role stencil's darn text, and the module docs the change touches.

Out:

- Editing or amending a claimed board entry's body: the record carries the correction instead.
- Loom's narrower gap, where `decision add` is refused before Discussion-Write has written the record.
- Moving the run verbs out of `lyx loom`: `run-verbs-shared` keeps them there for now.
- Splitting darn into Discussion-Write, Plan-Write and Webster rows: dropped by the operator.
- Migrating in-flight darn runs (see D11).

## Decisions

### D1 — The record format is one spec both writers point at

Decision: the decision-record and support-log format in `contracts/stencils/loom/loom-template-discussion.md`'s Step 5 moves verbatim into a new spec, `contracts/specs/loom-discussion-spec.md`, registered in `contracts/specs/specs.go` and read at call time from the told specs directory.
The Discussion stencil and the darn writer stencil each point at that spec and restate none of it.
The spec also states the two parts no writer authors: `### Added after Discussion` entries, appended only by `lyx loom decision add` and never edited or removed, and Go-written carry-over blocks under `## Open risks`.
The Discussion-Review rubric's writer/reviewer symmetry note names the spec beside the Discussion stencil.

Rationale: today the format contract lives only inside another producer's stencil, so the darn stencil has nothing to point at.

### D2 — Darn-Write writes the record before any code, and reads it on every spawn

Decision: the darn writer stencil gains a step between reading the task and making the change: write the decision record and support log in the spec's format, then run `lyx loom validate-discussion` until clean, before the first code edit.
The record holds the scope as the writer reads the board entry and the calls it makes; the support log's `## Interview` records the writer's reading of the entry and its self-made picks, since no interview happens, and `## Question ledger` marks each pick as an auto-pick.
The stencil's Step 1 reads the board entry together with the record's `Added after Discussion` entries, which win over the board body where they differ, on every spawn: the first, a rework after a PR-Gate or alignment reject, and a resume.
After its first clean validation the writer never edits the record, with one exception: restoring what a gate finding names.
A later spawn leaves an existing valid record alone; it writes the files only when they are absent, and fixes an invalid record in place, keeping every `Added after Discussion` entry and carry-over block, never rewriting it whole.
The `discussion` gate is added ahead of `verify` and `description` on Darn-Write, and ahead of `verify` on Darn-Burler-Fix, each with its budget renewed on `lyx loom resume` as the verify budget is.

Bound: the ordering "before code" is the stencil's instruction; the handoff gate proves only that a valid record exists when the writer or fixer hands off.
Carry-over blocks are kept by the stencil rule above, not by a mechanical check.

Rationale: the operator's direction keeps darn one combined agent and has it write the record in the same session; a record written first is there for the parent to amend while code is under way, and a re-spawn that read only the board body would undo the correction, the incident's failure.

### D3 — Go commits the discussion files; `decision add` commits on append

Decision: the `_lyx/discussion/` files sit on the fabric's records side, which the writer's own code commits do not reach, so Go commits them.
Darn-Write's Go-owned commit after the session, today the landing directory only, also commits the discussion directory through the same `CommitDiscussion` body `decision add` uses, whose pathspec also takes the parent-review round directories, absent on a darn run.
`lyx loom decision add` already commits the record it appends to, so a parent entry is in the record's history from the moment the verb succeeds, whether or not the writer's own record was committed before it.

Rationale: reusing the existing commit seam adds no new commit path.

### D4 — `decision add` works on a darn run, including while Darn-Write runs

Decision: `lyx loom decision add` on a darn worktree appends to the record once the record file exists, with no darn-specific refusal.
The refusal while Discussion-Write runs stays as it is and does not extend to Darn-Write: the darn writer owns its record only until its first clean validation.
The no-record way forward drops "a darn run never has one" and names that Darn-Write writes the record first, so on a darn run before that the call goes to the Darn-Write session as a message.
The refusal's spec row and test change with the text.

Bound: the verb now lets a parent append to a record while its writer session is live.
That is bounded by the unchanged caller checks (the run's recorded parent, the operator's unnamed shell, the run's own arbiter), one append-only entry per call under a labelled, dated heading, the discussion check re-run on append, the immediate commit (D3) and the history check (D5).
It takes no lock against the writer: a writer write after the append that drops the entry is caught at the next gate (D5), not at the write; the verb's restore on a finding restores the record it read, so a writer write inside that window can be lost; and the verb's commit can sweep in a record the writer has not finished, which then becomes history the D5 check holds later records to.
A record still half-written fails the discussion check on append, so the verb refuses and restores it untouched.

Rationale: the mid-run correction in the `webster-pack-last` incident arrived while the darn writer ran; refusing during that window would leave the gap open.

### D5 — A committed `Added after Discussion` entry cannot be dropped

Decision: `internal/discussionparser` gains one package function that checks every `Added after Discussion` heading present in a committed version of the record is still present in the record being checked.
The reference is the record's commits on the run's own branch since its merge base with the parent, so another task's record on the same path never counts.
`internal/fabricengine` reads that history, and `loomcli` wiring hands it to the gate as a told reader seam; `loomshed.NewDiscussionGate` does not open the fabric itself.
Three callers share the function: the `discussion` gate, `validate-discussion` and `decision add`, for every recipe; this extends `PATTERN-gate-self-check-parity`'s pairing of the discussion gate and `validate-discussion` to the new check, and its background's Pairs list changes with it.
Each finding names the dropped heading and quotes the entry from its last committed version, and its way forward is to restore that entry in the record with Edit and hand off again, a step every session may take without git.

Bound: the check compares headings, so it catches a dropped or retitled entry, not an edited body under an unchanged heading; the entry's body stays visible in the committed diff.
On loom it also forbids a `goto Discussion-Write` redesign or a parent-review rework from dropping or retitling an entry the parent added.

Rationale: the writer and Darn-Burler-Fix both write in the worktree after a parent entry lands, and the incident's revert came from a fixer; history rather than HEAD is the reference because a commit made outside Go cannot then erase an entry.
Applying it to every recipe keeps the gate free of a recipe branch, and a loom record follows the same append-only rule.

### D6 — The Darn review measures against the board entry as amended by the record

Decision: `loom-rubric-darn-review.md` and Darn-Burler-Review's `fasit` instructions name the measuring stick as the board entry together with the record's `Added after Discussion` entries, which win over the board body where they differ.
The writer's own record sections are a claim to check against the entry and those entries, as the change description is, never the measuring stick.
A diff that follows an entry the board body contradicts is not a scope finding.

Rationale: this departs from the board entry's wording, "holds it authoritative over the board body", by narrowing it to the record's parent-written entries.
The darn record is unreviewed before coding, so holding the writer's own sections authoritative would let the writer license its own scope; the parent's entries are the corrections the board body cannot carry.

### D7 — Darn-Bouncer carries open findings into the record

Decision: Darn-Bouncer gains `carry_over: Darn-Review`, so findings still open when the perch accepts land under the record's `## Open risks`, where the alignment agent reads them at PR-Gate.
The `CarryOver` seam is already built on the wiring path darn shares with loom.
The recipe comment that says darn has no record to carry into is removed.

Rationale: the alignment stencil's PR-Gate judgment reads open risks carried over from the run's last review; without this a darn run's accepted-but-open findings never reach it.

### D8 — Darn gains a PR-Align row

Decision: `contracts/recipes/darn-recipe.yaml` gains a `PR-Align` row, engine `PRAlign`, between Publish and PR-Gate: Publish's `on_done` becomes `PR-Align`, PR-Align's `on_done` is PR-Gate, it carries no `on_stuck` and no segment, as in loom.
The row uses the shared `landingAlignmentDeps` that darn's wiring already builds; the only change is that its prompt's plan directory renders `none` when the run's recipe is darn, decided by the recipe, not by whether the directory exists.
It reads loom.yaml's alignment keys (`skip_alignment`, `alignment_pr_approve`, the alignment model and timeout), so one hub setting governs alignment for both recipes; the doc comment on those keys in `internal/loomengine/config.go` says they govern darn runs too.
An agent reject writes the rejection record, and PR-Gate's Stuck sends it to Darn-Write through the existing pending-rejection path.

Bound: an agent approve now lands a darn run, and batten's Land row deploys, with no orch turn.
That is bounded exactly as in loom: `alignment_pr_approve`, the decision-snapshot and head checks, one applied agent reject per run, and `skip_alignment: true` or `alignment_pr_approve: false` leaving PR-Gate to the parent.

Rationale: the one input darn lacked was the decision record, which D2 supplies.

### D9 — The alignment stencil's PR-Gate judgment names no recipe row

Decision: `loom-template-alignment.md`'s `pr-gate` read and judge lists read the plan only when `{{.plan_dir}}` is not `none`, and measure the run against the board entry's goal and the record's decisions, its `Added after Discussion` entries winning over the board body, and its open risks, with no row name such as `Merge-Review`.

Rationale: the stencil's header claims it names no recipe row, and a darn PR-Gate has neither a plan nor a Merge-Review; one neutral text serves both recipes without a told fragment.

### D10 — Darn's Describe and the orch read the record

Decision: `landing-template-describe-darn.md` gains the record as a read source, so the description treats an `Added after Discussion` entry as the scope it implements, never as an unexplained deviation from the board body.
`orch-template-role.md` stops listing "any darn run" among runs with no record, tells the orch to record a darn correction through `decision add` once Darn-Write has written the record and to message the Darn-Write session before that, and also to message the live Darn-Write session whenever Darn-Write is the running row, since a live writer does not re-read its record.
It says a darn PR-Gate reaches the orch through alignment as a loom one does.

Rationale: both read the scope from the board body today and would otherwise contradict the corrected scope the review and alignment hold; a correction recorded without a message would reach the live writer only after the review the incident paid for.

### D11 — In-flight darn runs are restarted

Decision: no migration: an in-flight darn run lacks the record every row after Darn-Write now reads, and it is restarted.
A run resumed anyway halts at Darn-Bouncer's carry-over, which finds no record; that refusal's way forward names restarting the run.

Rationale: the darn recipe's header already restarts runs across row and edge changes; a single-operator system re-runs cheaply.

## Constraints

- `PATTERN-producer-pointer-rule`: the darn writer stencil, the Discussion stencil and the Describe stencil point at the discussion spec and never restate it.
- `PATTERN-stencil-ownership`: the new spec is read at call time from the told specs directory, never embedded into a producer.
- `PATTERN-sole-parsers`: the history check lives in `internal/discussionparser`.
- `PATTERN-fabric-git` and `PATTERN-told-geometry`: the record's committed history and every records-side commit go through `internal/fabricengine`, reached by the gate through a told seam.
- `PATTERN-gate-self-check-parity`: the gate, `validate-discussion` and `decision add` call one function for the history check (D5).
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: the changed `decision add` way forward, the new history-check refusal and D11's halt each name a way forward an agent may run, with its spec row and test.
- `PATTERN-edit-directive` and `PATTERN-parent-directive`: the darn writer stencil keeps rendering both directives.
- `PATTERN-test-economy` and `PATTERN-test-speed`: extend the existing recipe, decision and alignment tests rather than adding parallel ones; `cmd/lyx/testdata/test-budget.yaml` goes under `Edit:` on any card whose new top-level tests pass a package's row.
- Recipe row names are durable resume identities: no darn row is renamed, and `PR-Align` keeps loom's row name.
- `CLAUDE.md`'s docs rule: `internal/darnengine`, `internal/loomrecipe`, `internal/landingshed` and `internal/alignment` `doc.go`, the alignment keys' doc comment in `internal/loomengine/config.go`, and `docs/overview.md` if its execution stack names darn's rows, change in the same commits.

## Auto-mode assumptions

- Every decision is an auto-pick; D6 and D8 read the board entry loosest and are the ones to check.

## Open risks

- A writer or fixer that overwrites the record after a parent entry lands is caught only at its handoff gate or by the next `validate-discussion` or `decision add`, not at the moment of the write.
- The darn record is unreviewed before coding: a wrong scope reading in the writer's own sections is caught only by the Darn review or alignment, and corrected only by a parent `decision add` or a rework.
- Merges are expected with running and planned work: `twill-fork0` edits the Discussion stencil and the Discussion-Review fan, `orch-split` restructures the orch role stencil, `run-verbs-shared` may rename `lyx loom decision add` in every text naming it, `shed-tackler` changes darn's wiring, and `discussion-premise-cites` will edit the new spec and the discussion gate.

## Acceptance criteria

- The darn recipe's sequence runs Publish, PR-Align, PR-Gate, Finalize, and its routing projection and builder accept the new row with every review budget, including 0.
- A darn PR-Align with `skip_alignment: true` returns Done without spawning; with `alignment_pr_approve: false` an agent approve is left to the parent; an agent reject sends PR-Gate's Stuck to Darn-Write with the findings as the next spawn's rejection findings.
- The alignment prompt renders for a darn run with the plan directory `none`, and its `pr-gate` text names no recipe row.
- `lyx loom decision add --by parent` on a darn worktree with a valid record appends, commits and succeeds while Darn-Write is the running row; with no record it refuses with a way forward that no longer says a darn run never has one; on a half-written record it refuses and leaves the record as it was.
- The discussion gate, `validate-discussion` and `decision add` refuse a record that drops an `Added after Discussion` heading an earlier commit on the run's branch held, naming and quoting the entry, and pass a record that keeps every one; a heading only in history before the branch's merge base is ignored; the same holds on a loom record.
- Darn-Write's and Darn-Burler-Fix's handoffs fail their `discussion` gate when the record or support log is absent or invalid or drops a parent entry.
- A Darn-Write rework spawn after a reject, with a valid record holding parent entries, leaves the record unchanged and is told those entries as its scope.
- After a Darn-Write gate-exhaustion halt, `lyx loom resume` re-spawns it with a fresh `discussion` gate budget.
- Darn-Write's after-session commit includes the discussion directory.
- Darn-Bouncer's accept with open findings writes them under the record's `## Open risks`; on a run with no record it halts with a way forward naming a restart.
- The discussion spec is registered and seeded, the Discussion stencil's Step 5 and the darn writer stencil point at it, and the stencil-wording tests that pinned Step 5's text move with it.
- A Darn-Review round's prompt, the darn writer's prompt and darn's Describe prompt name the record's `Added after Discussion` entries as winning over the board body.

## Notes for the plan writer

- PR-Align's loom wiring is `landingAlignmentDeps` in `internal/loomcli/landingdeps.go`, built on the path darn shares; darn's own row wiring lives in `internal/loomcli/darnwiring.go`.
- `decision add`'s refusal texts are constants in `internal/loomcli/decision.go`; `commitDiscussion` in `internal/loomcli/wiring.go` is the shared commit body.
- The `discussion` gate is built in `internal/shedrecipe/entries_gate.go` through `loomshed.NewDiscussionGate`.
