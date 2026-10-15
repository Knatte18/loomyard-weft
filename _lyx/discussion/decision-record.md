# Decision record — discussion-relevant-list

## Goal

Plan-Write starts from a smaller context.
The loom Discussion writer already reads the code the task touches; it now writes that reading down as `_lyx/discussion/relevant.md`, a list of the files and members it read and found relevant, each with one line of why.
Plan-Write reads the list first and explores beyond it only where it needs to, instead of rediscovering the same code.

## Scope

In:

- A third Discussion-Write output, `relevant.md`, beside the decision record and the support log in `_lyx/discussion/`.
- Its format, pinned in `contracts/specs/loom-discussion-spec.md`.
- Its path, declared by a `loomengine` accessor beside `DiscussionDecisionRecord` and `DiscussionSupportLog`.
- Its format check, in `internal/discussionparser`, run by the loom discussion gate and by `lyx loom validate-discussion` for a loom run.
- The Discussion stencil (`loom-template-discussion.md`, and with it the chair stencil that includes it) asking for the file.
- The relaunch carrier per D1, so a relaunched writer writes it too: the Discussion host's twill values in `internal/loomengine/twill.go`, the new value in `internal/twilldirective`, and `contracts/stencils/twill/twill-relaunch.md`.
- The Plan stencil (`loom-template-plan.md`) reading it first.
- Doc comments: `internal/loomengine/plan.go`'s header loses "sole input", and `internal/discussionparser/doc.go` names the new accessor among the declarers and the new file among the formats it reads.

Out:

- A new role or research agent between Discussion and Plan.
- Glyphs, in `relevant.md` or in the discussion prose.
- Discussion-Review, its rubric, its fixer, the parent review brief and the Discussion-Write lens forks: none reviews `relevant.md`.
- Darn: its writer writes the decision record and the code itself, with no Plan step to read a list.
- The rework and prior-plan stencils: they keep reading the decision record only.
- `contracts/specs/loom-plan-spec.md`: it names nothing Plan reads from `_lyx/discussion/`, so it has no wording to change.
- Code changes to `tools/tokencount`.

## Decisions

### D1 — `relevant.md` is a loom Discussion-Write output

Decision: the Discussion-Write lead writes `_lyx/discussion/relevant.md` along with the decision record and the support log.
It is listed in the lead seat's `Outputs`, so the step's completion signal waits for it the way it waits for the other two, and the stencil's Step 5 and "What you may write" name three files.
Its path comes from a new `loomengine` accessor in the same family as `DiscussionDecisionRecord`, and no other package builds it.
The existing commit of the whole `_lyx/discussion/` directory carries it, so it is tracked, durable state.
A writer that changes the design in a later turn (a gate re-prompt or a parent-review re-prompt) may update the list in the same turn.
The Discussion lens forks are not told to review it: it stays out of the twill directive's reviewed files.
A relaunched writer still writes it.
The Discussion host's twill values (`discussionLensDirective` in `internal/loomengine/twill.go`) change `SpawnPoint` from "both output files" to "all three output files" per Step 5.
`twilldirective.Values` gains one host-supplied value naming the files a relaunched writer finishes, rendered by `contracts/stencils/twill/twill-relaunch.md` in place of "the reviewed files", which stays host-neutral.
`ReviewedFiles` is unchanged, so the forks still review two files.
The new marker is declared only in `twill-relaunch.md`, so the Plan host, which sets `WriterFork` and renders no relaunch part, supplies no value for it.

Rationale: the operator decided the Discussion step is extended, with no new role.
A required output means a run never reaches Plan with the list silently missing.

### D2 — Format: plain code names, one line of why each

Decision: `relevant.md` is a markdown bullet list, optionally under one H1 title.
Each top-level bullet opens with one backticked worktree-relative file path, followed by backticked member names in that file in plain Go spelling, unqualified (`PlanTable`, `Config.Validate`), wherever the writer read specific members, then ` — ` and one line saying why it is relevant.
No glyphs, no signatures, no `file:line` citations, no code excerpts.
The spec section for the file pins this, states that it is written by loom's Discussion-Write only (a darn writer does not write it), that the Discussion writer records only what it already read and never searches further to fill the list, and that it is a head start for Plan, never a completeness claim.
Plan's further exploration is D4's.

Rationale: the operator chose plain code names over glyphs, and no extra Discussion search on Plan's behalf.
Paths and member names stay inside the Discussion stencil's Step 2 bound, which forbids gathering signatures, `file:line` citations and interface shapes; excerpts would grow the Discussion context and go stale.
The darn writer reads the same spec, so the spec has to say the file is not its output.

### D3 — The check: a present list is well-formed

Decision: `internal/discussionparser`, the sole parser of `_lyx/discussion/` (`PATTERN-sole-parsers`), gets a separate check function for `relevant.md`, outside `Validate`.
It reports a finding when the file holds no entry; for any non-blank line that is neither the optional H1 nor a top-level entry bullet (nested bullets included); for an entry that does not open with a backticked path, whose path starts with `/`, a backslash, `~` or a drive letter and colon, or has a `..` segment between `/` or `\` separators, whatever the host OS, or that lacks the ` — ` reason or has nothing after it.
A missing file is not a finding, and a duplicate path is tolerated.
It does not check that a named path or member exists.
The loom discussion gate (`loomshed.NewDiscussionGate`, behind both the Discussion-Write row and the Discussion-Burler-Fix row) and the loom branch of `lyx loom validate-discussion` each call it besides `Validate`: one package function behind gate and self-check, per `PATTERN-gate-self-check-parity`.
`Validate` itself is unchanged, so `loomshed.ValidateDarnDiscussion` and `discussionparser.AppendDecision` (and with it `lyx loom decision add`) never see `relevant.md`.

Rationale: existence is enforced by D1's `Outputs`; the check only keeps a written list usable.
Tolerating absence keeps runs whose Discussion-Write finished before this lands passing every later gate and `decision add`, and keeps the fix-step gate, whose fixer never touches the list, from ever failing on it.
Bound: the path rule keeps an entry's backticked path inside the worktree; the reason text is prose and unchecked; a tracked symlink inside the worktree needs no rule, since it is repository content the list does not introduce.
The list is advisory, so a stale name costs Plan one failed read.
A writer that skips `relevant.md` passes `lyx loom validate-discussion` and is caught by shuttle's held turn end, which notifies the parent on an autonomous run and holds an interactive run for the operator, bounded by the Discussion timeout.
No gate re-prompt names the missing file, since the gate runs only once every output exists.

### D4 — Plan-Write reads the list first, when it exists

Decision: the Plan stencil's Step 1 names `relevant.md` beside the decision record: read the decision record, then the list, then the code the list names that the plan needs, then explore further as needed.
The decision record stays the only source of design and scope; the list never adds scope.
A missing `relevant.md` is not an error for Plan: it explores as it does today.
The discussion spec's "the plan writer's sole input" wording changes to say the decision record is the design input and `relevant.md` the code head start, and Plan reads nothing else from the discussion directory.

Rationale: the list is where the context saving comes from.
A run whose Discussion finished before this lands has no list, so Plan must tolerate its absence.
Bound: Plan reads one more file and nothing else from `_lyx/discussion/`; the support log and the board stay unread by Plan.

### D5 — No reviewer reviews the list

Decision: Discussion-Review's reviewer and fixer, the parent review and the Discussion-Write lens forks are not pointed at `relevant.md`, and their stencils, briefs and rubric do not change.

Rationale: the operator decided Discussion-Review does not review the list as design, so the review does not grow.
A list made stale by a review fix costs Plan at most an extra read, since Plan treats it as a head start.

### D6 — Code names go in `relevant.md`, not in `## Notes for the plan writer`

Decision: the discussion spec states that the list of code the writer read belongs in `relevant.md`; `## Notes for the plan writer` keeps design hints only.
The spec's description of that optional section is reworded to design hints for the plan writer, never a completeness requirement, and the head-start wording lives only in the `relevant.md` section.
A hint may still name the symbol it is about, so no constraint the plan writer needs is pushed out of the record.

Rationale: one place for the code list keeps the decision record lean and avoids the same names in two files.

## Constraints

- `PATTERN-sole-parsers`: only `internal/discussionparser` parses `relevant.md`; it stays a stdlib-only leaf and takes told paths.
- `PATTERN-gate-self-check-parity`: the gate and the verb call the one new check function, landed in this task.
- `PATTERN-cwd-resolution` and `PATTERN-told-geometry`: the path is declared once, by the `loomengine` accessor; `discussionparser` declares no location.
- `PATTERN-durable-vs-ephemeral-state`: the file is tracked, under `_lyx/discussion/`.
- `PATTERN-producer-pointer-rule`: the Discussion and Plan stencils point at the discussion spec for the format and never restate it.
- The running sibling `engine-model-choice` rewrites `DiscussionTable`'s model resolution in `discussiontable.go`; this task only adds an entry to the lead seat's `Outputs` there and leaves `resolveModelChoice` and the seat literal's shape alone.

## Auto-mode assumptions

- The list is required for every loom Discussion-Write run, interactive or autonomous, with or without advisors or lenses.
- At least one entry is required: every loom Discussion reads some file, a doc or a stencil if no Go code.

## Open risks

- Plan-Write re-explores anyway and the context does not shrink.
- The list misses code the plan needed and a card goes wrong; Plan's freedom to read more limits it.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d1-plan-host-relaunch-value (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-1-review.md; fixer report: _lyx/reviews/plan/round-1-fixer-report.md.
  - relevant-path-wire-assignment-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Plan-Review -->

<!-- lyx:carry-over Webster-Review -->
- Webster-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/webster/round-1-review.md; fixer report: _lyx/reviews/webster/round-1-fixer-report.md.
  - relevantpath-guard-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Webster-Review -->

## Acceptance criteria

- A loom Discussion-Write lead seat lists `relevant.md` among its outputs.
- The `relevant.md` check passes a well-formed list and a missing file, and reports a finding for each malformed case D3 names.
- The loom discussion gate and `lyx loom validate-discussion` on a loom run report the check's findings, and pass a run with no `relevant.md`.
- `lyx loom validate-discussion` on a darn run and `lyx loom decision add` pass with no `relevant.md` and ignore a malformed one.
- Stencil and spec wording (Discussion Step 5 and "What you may write", the relaunch wording, Plan Step 1, the spec's format section and head-start rule) is checked by review, not by prose assertions in tests.
