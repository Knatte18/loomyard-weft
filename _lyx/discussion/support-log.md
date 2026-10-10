# Support log: Discussion-Write on MultiLLMProducer

## Interview

The session ran autonomously, so every batch below was put to the codebase and the board and settled by the writer.

Turn 1, scope and the problem.
Is a multi-seat Discussion the right thing to build, or should the single agent stay?
The board entry records the operator's call (2026-10-08) that a good Discussion protocol is worth the tokens, and the producer task shipped the seat engine with Discussion-Write as its first consumer.
Settled: build it, and make zero advisors the single-agent case so one code path serves both.

Turn 2, where the seats are declared.
The producer task allows the recipe row or one Go table.
The discussion stencil needs per-run values (slug, the two paths, mode rules, the pattern and friction directives) that a recipe `tokens` map cannot carry, the same reason the row never used the generic `SingleLLM` entry.
Settled: a Go table in `internal/loomengine`, told to the entry through a per-call closure that replaces the discussion spec closure.

Turn 3, the producer's missing seams.
The DiscussionWrite entry today wires the parent-review store's `PrepareRound` as the single-LLM producer's fresh-spawn preparation, and the spec is a per-call closure.
`MultiLLMProducer` has neither.
Settled: add both through a second constructor, keeping the shipped constructor unchanged for the sibling Burler task.
The preparation returns only an error: the one preparation this task wires always returns an empty amendment, and carrying one into a seat's prompt would need a seat engine field no caller uses.

Turn 4, config keys and models.
The operator named the models per seat: chair `opus[high]`, advisor `sonnet[high]`, after a step down from `fable[high]`.
Settled: `discussion` stays the chair's key, `discussion_advisors` is a list of model-specs whose length is the advisor count, empty legal, template default one `sonnet[high]`.

Turn 5, how many advisors and what each is for.
Options were the same brief per seat, or distinct reader and challenger roles.
Settled: one scout brief per seat, writing its study early and answering with evidence, plus an unprompted conflict message; the challenger's value is covered by that message, the parent review and the Discussion-Review perch.

Turn 6, when the chair ends and what an advisor's output looks like.
Settled: the chair ends exactly as today, after both files and the self-check; the advisor writes a notes file under `_lyx/discussion/`, committed with the discussion, read by no parser and never by Plan, and the decision record carries no advisor section.

Turn 7, routing and the operator's turns.
Settled: every message to the producer goes to the chair; the operator's interactive turns go to the chair alone, since the seat engine makes only the chair interactive; the parent-review delivery and rejects ride the chair's gate as today.

Turn 8, edge cases.
Advisor start failure, death, timeout, chair death, driver crash, parent-review reject mid-run: all are shipped seat-engine behaviour, listed in D9 so the plan does not re-derive them.
An in-flight run under the old binary: restarted, following the recipe header's precedent.

Turn 9, testing.
Settled: untagged tests over the table builder, the entry, the producer's two seams, the stencils' claims and markers, and the recipe's built row list; no new LLM-tier test.

## Rejected alternatives

- A generic `MultiLLM` recipe row for Discussion-Write with the seats in YAML.
  Rejected: per-run marker values and the `discussion` model key cannot be expressed in a static row, which is the recorded reason the row never used the generic `SingleLLM` entry; it would also need an Env-side value source and a model indirection the recipe grammar lacks.
- Keeping the single-agent spec path beside the seat path, switched by advisor count.
  Rejected: two prompt composers for one role drift, and the N=0 table already is the single agent.
- Renaming the row's engine to `MultiLLM`.
  Rejected: the engine name is the row's contract in the row-to-engine table, the coverage guard and the gate-quiescence writer list; nothing is gained by churning them.
- Two advisor briefs, a code reader and a design challenger.
  Rejected for now as YAGNI: the scout's unprompted conflict message, the parent review and the Discussion-Review perch cover the challenge, and a second brief doubles the stencil surface before one advisor has been measured.
- A separate advisor-count key beside the model list.
  Rejected: the list length is the count; a second key can disagree with it.
- Advisor notes under `.lyx/` or in the review run root.
  Rejected: the seat engine resolves outputs under the worktree root and the discussion commit already covers `_lyx/discussion/`; an ephemeral file would be lost on the commit the chair's outputs ride.
- An advisor section in the decision record.
  Rejected: the record's section set is pinned by the discussion parser and the record is Plan's sole input; advisor reliance belongs in the support log's interview.
- A preparation with the single-LLM shape, its amendment appended to the chair's prompt, for `plan-multi-llm` to reuse.
  Rejected: this task's only preparation returns an empty amendment, the append needs a new seat engine field, and `plan-multi-llm` is out of scope; that task adds the amendment path when it has a caller.
- Leaving the two optional directives out of the table when empty, or filling them with a placeholder.
  Rejected: the stencil fill refuses a missing key, and a placeholder puts text in the prompt that today's path does not; a listed optional value keeps today's rendering.
- Stopping the advisors before the chair's gate runs, so no advisor edit can land while the gate waits.
  Rejected: a parent-review reject re-prompts the chair inside the gate, and the advisors must still be up to answer the rework's questions (D9); it would also need a seat engine change this task's additive rule does not cover.
- An `llm`-tier test for the discussion table.
  Rejected: the seat engine's smoke already proves the channel, and no gate runs the tier.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Build a multi-seat Discussion at all? Resolved, auto-pick: yes, per the operator's recorded call on the board entry.
- Q2 Seats in the recipe row or a Go table? Resolved, auto-pick: Go table in `loomengine`, per-call closure to the entry.
- Q3 Keep the `DiscussionWrite` engine name? Resolved, auto-pick: yes.
- Q4 How does the parent-review preparation reach the fresh start? Resolved, auto-pick: a fresh-spawn preparation seam on `MultiLLMProducer` that returns only an error.
- Q5 Config keys? Resolved, auto-pick: `discussion` for the chair, `discussion_advisors` list for the advisors, empty legal.
- Q6 Template default for `discussion_advisors`? Resolved, auto-pick: one `sonnet[high]`; the chair's step-down is the operator's edit.
- Q7 How many advisors, which roles? Resolved, auto-pick: one scout brief, count from config.
- Q8 Advisor output file? Resolved, auto-pick: `_lyx/discussion/advisor-<n>.md`, committed, unparsed.
- Q9 Advisor text in the decision record? Resolved, auto-pick: none; reliance is recorded in the support log's interview and, for autonomous picks, the question ledger, both by strand name.
- Q10 Where does the advisor-aware wording live? Resolved, auto-pick: the Go-held autonomous `mode_rules` text and the chair stencil's interview step; no stencil asks the operator.
- Q11 Operator's interactive turns? Resolved, auto-pick: chair alone.
- Q12 Advisor skills? Resolved, auto-pick: `scribe:prose` only.
- Q13 Advisors message each other? Resolved, auto-pick: no, the shipped advisor directive forbids it.
- Q14 In-flight runs under the old binary? Resolved, auto-pick: restarted, not migrated.
- Q15 New LLM-tier test? Resolved, auto-pick: no.
- Q16 Does the token report need a change for the new roles? Resolved from the code: no, it keys by the strand's last segment with the trailing number dropped.
- Q17 Overlap with `burler-multi-llm`? Open risk, mitigated: additive changes only in the shared packages.
