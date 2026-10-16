# Decision record — orch-intake-fork

## Goal

The orch learns whether the GitHub inbox holds anything from issue numbers and titles alone, and a one-shot fork does the intake itself, so no issue body enters the orch's context.

## Scope

In:

- `lyx board intake list` prints numbers and titles by default; the full view moves behind a flag.
- `lyx board intake import` prints a short result instead of the whole entry.
- The orch role stencil (`contracts/stencils/orch/orch-template-role.md`): the orch checks the inbox with the compact list and hands a non-empty inbox to one one-shot intake fork.
- The `ly:board` skill's Intake section, which the fork follows, matches the new output and flag.
- `internal/boardcli/doc.go` states the new list and import output, in the same commit as the code.

Out:

- `intake close`, its payload and output.
- `boardengine`'s import, fold and labelling rules, and `selfreportengine`'s GitHub calls.
- Moving the inbox check out of the orch into a Go run driver, which `orch-split` covers.
- Deriving labels from a diff, which `labels-from-diff` covers.

## Decisions

### D1 — `intake list` is compact by default

`lyx board intake list` prints `{"issues":[{"number":…,"title":…}]}`, one object per open inbox issue no entry records, pull requests excluded, an empty inbox as `{"issues":[]}`.
A `--full` flag prints today's view: number, title, body, labels, url, comments and created_at.
Filtering is the same in both modes, and an error prints the usual error envelope in both.

Rationale: the cheap form is the default, so an orch that runs the bare command never pulls a body into its context, and the fork that needs bodies asks for them.

### D2 — `intake import` prints a short result

A successful import prints `{"slug":…,"issue":…,"labels":[…],"dropped":[…]}`: the written entry's slug, the issue number, the entry's labels after the write and the issue labels not carried over.
A fold prints the same, with `slug` the entry folded into.
When the board write succeeds but the GitHub comment or close fails, the error envelope carries the same four fields plus the `close` payload to re-run, in place of today's `entry`.
The already-recorded no-op keeps its `recorded` output.

Rationale: the caller needs to know where the issue went and with which labels, not the body it just read; `lyx board get` shows the entry when anyone wants it.

### D3 — The orch checks the inbox and forks intake

The orch role stencil states the rule in Where a finding goes, and After landing points to it:

- At every wake, the `landed` notice included, the orch runs `lyx board intake list` and reads only its numbers and titles.
  It never runs `--full` or `intake import` itself.
- An empty list ends the check.
- An error envelope is neither empty nor non-empty: the orch starts no fork and handles the error as a lyx bug under Where a finding goes.
- A non-empty list goes to one one-shot fork, unless the orch knows an intake fork is open, in which case it starts no second one.
- An open intake fork is not named in the orch note: the inbox is its own state, so a fork lost to a context cycle leaves its issues open and a later wake's check forks again.
  A second fork live beside the first after a cycle is harmless, since the CLI settles each issue once: a second import of a recorded issue prints `recorded`, a close of an imported issue is refused as closed, and an import of a closed issue is refused.
- A `failed` line in the intake fork's report is handled as a lyx bug under Where a finding goes, naming the issue and, for a failed close, the `close` payload.

Rationale: the operator's decision (2026-10-16); a fork, not a fresh agent, because it inherits the board policy and the operator's settled decisions.

### D4 — What the intake fork may do

The stencil states the outcomes and bounds below and points to `ly:board` for the commands (D5); it does not restate the commands' mechanics.

The fork reads the full list and takes each issue to one of three outcomes:

- A new note, with exactly one type label and the label of every module the issue touches.
- A fold into an existing entry: only when the issue is essentially the same finding as that entry, or a correction to its subject, per the orch's board policy; the fork then gives the entry any module label the issue adds, since every entry carries the labels of the modules it touches.
  The board refuses a fold into a run-held entry, and such an issue becomes a new note.
- A close as noise: only an issue that adds nothing to the recorded issue or entry it duplicates, or an issue with no actionable content, with a reason naming which; a duplicate that adds anything to the entry is folded into it, or becomes a new note when the entry is run-held.

When an import's board write succeeds but its GitHub close fails, the fork re-runs the error envelope's `close` payload once, since a recorded issue no longer shows in `list` and nothing else would close it.
The re-run accepts a second pointer comment when the first one posted.
A second failure gives the issue a `failed` line carrying the error and the `close` payload.

Bound: the fork writes the board only through `lyx board intake` and a `lyx board upsert` that writes only the labels of an entry it just folded an issue into, keeping every label the entry carries and adding the issue's module labels.
It edits no file, sends no message, and merges, promotes, starts or re-scopes nothing.
It returns one line per issue: the number, then the new slug, the entry folded into, `closed`, or `failed`, then the labels, the close reason, or the error.

Rationale: the operator's decision names the three outcomes and module labels; these bounds keep the fork's writes to what intake needs, since every other board change stays the operator's call.

### D5 — The `ly:board` skill matches

The skill's Intake section says `list` is compact and `--full` shows bodies.
It says a new note gets its type and module labels through `import`'s `labels`, replacing the line that adds module labels with `upsert` afterwards.
It says a label `upsert` on a folded entry carries the entry's whole label list.

Rationale: the fork follows this skill, and D1 and D2 make its Intake section wrong as it stands.

## Constraints

- `PATTERN-cli-cobra`: `--full` is a flag on the existing cobra `list` command.
- `PATTERN-role-skills-typed`: the stencil asks no session to load a skill; the fork inherits the orch's loaded `ly:board`.
- `PATTERN-no-denied-recovery`: both edited prose files, the orch role stencil and the `ly:board` skill, name only commands the agents' settings allow.
- `PATTERN-test-economy`: no test pins the stencil's or skill's prose; review guards the wording.

## Auto-mode assumptions

- D1's compact default and its `--full` name, and D3's every-wake cadence, are self-picks: ledger Q1, Q2 and Q4.

## Open risks

- A running orch keeps its rendered role file and loaded `ly:board` skill until its next context cycle, so after the deploy it does intake the old way, at the old token cost, until then.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-1-review.md; fixer report: _lyx/reviews/discussion/round-1-fixer-report.md.
  - d4-close-noise-fold-overlap (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- `intake list` with no flag prints only number and title per unrecorded open issue, with recorded issues and pull requests left out.
- `intake list --full` prints today's fields for the same set.
- Both modes print `{"issues":[]}` for an empty inbox.
- A new-note import prints slug, issue, labels and dropped, and no body.
- A fold prints the target entry's slug and its labels after the fold.
- A failed close after a written import returns the four short fields and the `close` payload, and no `entry`.
- An already-recorded issue still prints `recorded` and touches nothing on GitHub.
- By review: the orch role stencil states D3's check, its error case and the one-fork rule, and D4's outcomes, bounds, close re-run and return line; the `ly:board` skill states D5; and `internal/boardcli/doc.go` states D1's list output, the `--full` flag and D2's import output.

## Notes for the plan writer

- The test for `intakeList` already seeds recorded, closed and pull-request issues and can cover both modes.
