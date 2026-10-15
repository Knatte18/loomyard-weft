# Decision record — twb-d0-sa

## Goal

A scope correction made on a running darn task reaches every later row of the run, and a darn pull request can be settled by the alignment agent at PR-Gate the way a loom one is.
Darn stays one combined writer session; it gains a short decision record written before any code, `lyx loom decision add` works on it, the Darn review and the alignment agent treat the parent's recorded corrections as authoritative over the board body, and the darn recipe gains a `PR-Align` row.

## Scope

In:

- Darn-Write writes loom's two discussion files, `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`, before it edits any code.
- The record is committed through the fabric while Darn-Write still runs, before any code commit.
- `lyx loom decision add` works on a darn run once the record is committed, including while Darn-Write runs.
- A guard that no `### Added after Discussion` entry the run committed is ever dropped on a darn run.
- The Darn-Write row gains the `discussion` gate; Darn-Bouncer gains `carry_over: Darn-Review`.
- The Darn-Review rubric and the Darn-Burler-Review row's `fasit` instructions measure against the board entry as amended by the record's `Added after Discussion` entries.
- A `PR-Align` row in `contracts/recipes/darn-recipe.yaml` between Publish and PR-Gate, wired for a run with no plan.
- The alignment stencil's `pr-gate` measuring stick becomes a recipe-told fragment.
- The orch role stencil, and every way-forward text and refusal-spec row that says a darn run has no record.
- Docs in the same commits: the touched packages' `doc.go`, the recipe header comments, and `docs/overview.md` only if its execution stack names darn's rows.

Out:

- Editing a claimed board entry's body, or appending amendments to it; the record is the one place a correction lives.
- Turning darn into a loom without perches (Discussion-Write, Plan-Write, Webster in turn); the board entry drops it.
- A review of the darn record before coding; Darn has no Discussion-Review perch and gains none.
- Moving `decision add` or the other run verbs out of `lyx loom`; that is `run-verbs-shared`, which keeps them under `lyx loom` for now.
- Loom's own pre-record gap (`decision add` refused before Discussion-Write has written the record); loom keeps it.
- Darn-specific alignment config keys.
- Darn's Describe stencil; it describes the diff and is unchanged.
- Migrating in-flight darn runs.

## Decisions

### D1 — Darn-Write writes the record first, in loom's format

Darn-Write's first work, after reading the board entry and `CLAUDE.md`, is the two discussion files: the scope as it reads the entry and the calls it makes, short, in the format `loom-discussion-spec.md` defines, with no interview and no lens forks.
The darn writer stencil points at that spec and restates none of it.
It runs `lyx loom validate-discussion` until clean before editing any code.
A later spawn of the row (a verify re-prompt, prior work or a PR-Gate rejection) rewrites the files only when they are absent or invalid, and a rewrite keeps every `Added after Discussion` entry verbatim.
On a rejection spawn whose findings show its own scope reading was wrong, the writer may revise its own sections, never an `Added after Discussion` entry.

Rationale: one session already explores the code for the change; writing its reading down costs a few minutes and gives every later row and the parent one shared statement to amend.

### D2 — A clean `validate-discussion` on a darn run at Darn-Write commits the record

`validate-discussion` reads the run's seed recipe and status.
On a darn run whose current producer is Darn-Write, in any state (running, or halted there for the parent), a clean pass commits the discussion directory through the existing `commitDiscussion` fabric commit, so the record is committed before any code exists.
Bound: the commit happens only on a darn run, only at Darn-Write, only on a clean validation, and only over the discussion pathspec `commitDiscussion` already stages; an unchanged directory commits nothing; on a loom run and at any other row the verb stays read-only.
A failed commit is refused with the way forward `decision add` already gives for its own failed commit.
`PATTERN-fabric-git` says the weft commit is Go calling the engine at a boundary the run controls; here Go makes the commit inside a lyx verb under the bound above, the same shape as `decision add`'s commit, and the agent never runs git on the weft.

Rationale: `_lyx/discussion/` is committed through `fabricengine`, never the writer's own `git commit`, and the writer has no other sanctioned way to reach that commit mid-session.

### D3 — `decision add` works on a darn run once the record is committed

`decision add` reads the run's seed recipe.
On a darn run it is accepted at any row, Darn-Write included, provided the record is committed and its working copy equals its committed copy.
It refuses an absent record and an uncommitted record with the way forward: re-run once Darn-Write has validated and committed its record, and until then message the Darn-Write session.
It refuses a working copy that differs from the committed copy: at Darn-Write the way forward is a re-run once the writer validates; past Darn-Write it is the existing commit-failure way forward (`lyx fabric commit`, which the orch runs for a session lyx refuses it from).
Calls on one record are serialized under a lock in the run's ephemeral tree, so two concurrent appends cannot lose an entry.
After its commit, the verb confirms the committed record holds its own entry, and refuses with a re-run way forward when it does not.
The existing refusal while Discussion-Write runs stays loom's alone, and `decisionWayNoRecord` drops "a darn run never has one".
Bound: on darn the verb's window is from the writer's first clean validation to the run's end; one append per call under the existing caller check, including the run's own arbiter label, and the append-only heading, in a committed diff.
Before the record is committed a correction still reaches only the Darn-Write session; that gap is the minutes before the writer's first clean validation.
At Darn-Bouncer the existing transient "carry-over record changed" refusal now also covers a darn settle racing the verb.

Rationale: the darn correction that motivated the task arrived while Darn-Write ran, so the verb must work during that row; requiring a committed, unchanged record keeps it from racing a writer mid-edit.

### D4 — No `Added after Discussion` entry is ever dropped on a darn run

On a darn run, the Darn-Write `discussion` gate, a clean `validate-discussion` and `decision add` each refuse a record that lacks an `Added after Discussion` heading any commit of the record since the task branch's merge base with its recorded parent held.
The lower bound keeps another task's record history inherited from the parent's records branch out of the comparison.
The comparison is a `discussionparser` function over the parsed headings (`PATTERN-sole-parsers`); the history read is a `fabricengine` function (`PATTERN-fabric-git`).
Under `PATTERN-gate-self-check-parity`, the gate closure and `validate-discussion` call one shared function for this check, and the entry's background gains that pair.
The refusal prints each dropped entry's full committed text, so the recipient restores it with Edit and needs no git read of the weft (`PATTERN-no-denied-recovery`).

Rationale: on a darn run the writer and the parent can both touch the record while Darn-Write runs, and the spec already says a writer never removes such an entry; this makes that mechanical where the race exists.

### D5 — The Darn review measures against the amended board entry

The Darn-Review rubric names the measuring stick as the board entry as amended by the record's `Added after Discussion` entries, which win over the board body where they differ, and the Darn-Burler-Review `fasit` instructions point at the rubric for it.
The writer's own record sections are a claim to check against the diff, like the change description, never the measuring stick.
A diff that departs from the board body in a way the change description attributes to a parent correction with no matching `Added after Discussion` entry is a MEDIUM `decision`-class finding whose fix leaves the code unchanged and names the parent's `decision add` as the way to record it.
This narrows the board entry's "holds the record authoritative over the board body" to the record's `Added after Discussion` entries.

Rationale: today the review reads the claimed board body live with `lyx board get`, so a correction made anywhere else is invisible to it, which is what reverted the `webster-pack-last` correction; the writer's own sections are unreviewed, so they cannot outrank the board.

### D6 — Darn-Bouncer carries open findings into the record

Darn-Bouncer gains `carry_over: Darn-Review`, so findings still open when the perch accepts land under the record's `## Open risks` through the existing carry-over seam, as loom's perches do.

Rationale: PR-Align and the parent read the record; an accepted-but-open finding is what they most need to see.

### D7 — The darn recipe gains PR-Align

A `PR-Align` row with engine `PRAlign` sits between Publish and PR-Gate, outside the PR-Review segment, as in loom.
Publish's `on_done` names it, and Merge-In's reroute and a PR-Gate rejection re-enter it with the new head, as in loom.
Its prompt inputs are the run's own record and support log paths; a darn run passes no plan directory, and the alignment prompt fill renders an empty plan directory as `none`, a fill it does not make today.
It reads loom.yaml's alignment keys, as darn's perch already reads loom.yaml's Webster-Review model keys, so `skip_alignment: true` and `alignment_pr_approve: false` leave a darn PR-Gate to the parent.
Bound: an agent approve now lands a darn run and batten's Land row deploys with no orch turn; that is bounded by `alignment_pr_approve`, the decision snapshot and head checks, and one agent reject per run, exactly as on loom.
No row is renamed.

Rationale: the one PR-Align input darn lacked was the decision record, which D1 supplies.

### D8 — The alignment stencil's `pr-gate` measuring stick is a told fragment

The alignment stencil names no recipe row; what `pr-gate` measures the diff against and what counts as a scope deviation come from one told marker the wiring fills per recipe.
Loom's fragment keeps today's text verbatim.
Darn's fragment names the board entry as amended by its `Added after Discussion` entries, which win over the body, and the carried-over `Darn-Review` open risks; the writer's own sections are a claim.
On a darn run, an open risk or a diff that rests on a parent correction with no recorded entry is an `escalate`, never an approve or a reject.

Rationale: the darn record's writer sections are unreviewed, so the same "recorded decisions" clause would let an unreviewed reading outrank the board; a told fragment keeps one stencil (`PATTERN-stencil-ownership`).

### D9 — The orch records corrections in the record

The orch role stencil says a correction to a running task is recorded with `lyx loom decision add --by parent` whenever the run has a record, darn included, and that a darn PR-Gate reaches the orch as an alignment escalation or note.

Rationale: the orch is the parent that makes corrections.

### D10 — In-flight darn runs are restarted

Every row after Darn-Write assumes the record exists, so a darn run in flight when this lands is restarted, not migrated.
A darn PR-Align whose record is absent spawns no agent and leaves PR-Gate to the parent with an alignment note naming the missing record and the restart.

Rationale: the restart norm the recipe header comments state; the PR-Align guard covers the one row that would otherwise judge without a record, when `review_max_bounces: 0` skips the perch's carry-over.

## Constraints

- `PATTERN-fabric-git`: every commit and history read of the record goes through `internal/fabricengine`; D2 names the boundary clause.
- `PATTERN-sole-parsers`: the kept-entries comparison lives in `discussionparser`.
- `PATTERN-gate-self-check-parity`: D4's check is one function the gate and `validate-discussion` share.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: each new refusal in D2, D3, D4 and D10 names its way forward, lands its `contracts/specs/refusal-spec.md` row and a test, and names no command the agents' settings deny; the existing `decision: no decision record` row changes with `decisionWayNoRecord`.
- `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule`: the darn writer stencil points at `loom-discussion-spec.md` and paraphrases none of it; stencils are read at call time.
- `PATTERN-parent-directive` and `PATTERN-edit-directive`: the darn writer stencil keeps rendering both.
- `PATTERN-shed-recipe-registry`: the new darn row uses the existing `PRAlign` constructor.
- `PATTERN-durable-vs-ephemeral-state`: D3's lock lives under `.lyx`.
- Row names are durable resume identities pinned against `internal/loomshed`'s `Name*` constants.
- Docs land in the same commits as the code (`CLAUDE.md`), and a package past its `cmd/lyx/testdata/test-budget.yaml` row lists that file under `Edit:`.

## Auto-mode assumptions

- The darn record is short: the writer's reading and calls, no interview turns.
- D3 makes `decision add` work once the record is committed, not merely once it exists, as the board entry words it; the gap is the minutes before the writer's first clean validation.
- D2's commit side effect on `validate-discussion` is preferred over a new verb.

## Open risks

- `shed-tackler` would give darn an arbiter, whose `--by arbiter` call then reaches D3's darn window, Darn-Write included; D3's bound covers that label, and nothing else in this record assumes darn has no arbiter.
- `run-verbs-shared` builds on this task's darn record and may later move `decision add`.
- `orch-split` restructures the orch role stencil D9 edits and plans to take PR-Gates off the orch, which changes where D7's escalations land.
- `loom-model-key-names` may rename loom.yaml keys; D7 reads the alignment keys by their current names.

## Acceptance criteria

- The darn recipe builds with a `PR-Align` row between Publish and PR-Gate, a `discussion` gate on Darn-Write, and `carry_over: Darn-Review` on Darn-Bouncer; the row-name pins and routing tests cover the new row, and with `review_max_bounces: 0` Darn-Write still routes to Merge-In and PR-Align still runs.
- A darn Darn-Write handoff with no record, or an invalid one, is held by the `discussion` gate and re-prompted.
- `validate-discussion` on a darn run at Darn-Write, running or halted, commits the discussion directory on a clean pass and commits nothing on a finding; at another darn row and on a loom run it commits nothing.
- `decision add` on a darn run: succeeds while Darn-Write runs with a committed, unchanged record; refuses with its way forward for an absent record, an uncommitted record, and a working copy that differs from the committed one, with the past-Darn-Write way forward after an earlier failed commit; refuses when its own entry is missing after the commit; two concurrent calls both land their entries.
- A darn record that drops an `Added after Discussion` heading a commit since the merge base held is refused by the gate, by `validate-discussion` and by `decision add`, even when the latest commit already dropped it, and the refusal prints the dropped entry's text; a fresh darn record passes when the parent's records history holds another task's record with such entries.
- The writer-overwrite race: the parent appends during Darn-Write, the writer's later rewrite drops the entry, and the gate re-prompts the writer until it is restored.
- The incident end to end: a parent correction recorded during Darn-Write reaches the Darn-Review prompt as the measuring stick, and the rubric states the amended board entry; an unrecorded correction the description names is a MEDIUM `decision` finding.
- A loom run's `decision add`, `validate-discussion` and alignment prompt behave as before.
- The alignment prompt for a darn run renders the darn fragment and `none` for the plan directory; a loom run renders today's text.
- PR-Align on a darn run: with alignment off it is Done with nothing spawned; an agent approve with `alignment_pr_approve: true` writes the approval PR-Gate reads; with it false the approval is left to the parent with a note; an agent reject sends PR-Gate back to Darn-Write, the rework re-enters PR-Align with the new head, and a second agent reject goes to the parent; an absent record leaves PR-Gate to the parent with a note.

## Notes for the plan writer

- The Darn review's scope source is the rubric and the `fasit` instructions in `contracts/recipes/darn-recipe.yaml`.
- `internal/loomcli/decision.go` holds the writer-owns-record refusal and `decisionWayNoRecord`; `commitDiscussion` in `internal/loomcli/wiring.go` is the pathspec-scoped commit D2 and D3 reuse.
- `landingAlignmentDeps` in `internal/loomcli/landingdeps.go` builds PR-Align's seams for any worktree and always passes a plan directory today; `wireDarn` in `internal/loomcli/darnwiring.go` is where darn-only wiring lives.
- `discussionparser.ParseAddedDecisions` already parses the headings D4 compares.
- Stencil-wording tests pin the darn writer, alignment and rubric text, so the change moves those pins.
