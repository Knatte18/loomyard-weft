# burler-cost — support log

## Interview

Autonomous mode: no operator answered; every decision below is a self-pick recorded in the question ledger.

1. Read the board entry `burler-cost` and the entries touching it: `two-agent-producer`, `discussion-advisor`, `cache-ttl-per-role`, `convergence-plan`, `extra-agent-strand`, `bugfix-sessions`, `test-tiers`.
   `two-agent-producer` names this task's split as one of its two consumers and places two agents inside one producer, never as two shed producers.
2. Read `burlerengine` (doc, engine), the round stencils, `shedrecipe/entries_burler.go`, `shedadapters/burler.go`, `loomengine/review.go`, the loom recipe's Burler rows and shuttle's run and hold surfaces.
   Findings: one round is one shuttle run under role `burler`; no recipe row names a cluster fan; shuttle already offers `StartGated` plus `Wait`; an autonomous turn end without outputs is held and notifies the parent; `RunOpts` is documented as per-round run-tuning.
3. Scope question: build the generic two-agent producer or a Burler-specific split.
   Picked Burler-specific (D1): the round needs a one-way handoff, the advisor needs a dialogue, and YAGNI rules out a generic layer for a second consumer that is still a note.
4. Handoff question: how the fixer learns the review is ready.
   Picked a Go-written marker plus a polling `lyx burler await-review` verb (D5).
5. Validator question: what "grain of salt" lets the fixer do.
   Picked an evidence-bound dispute section (D8), with a `PATTERN-review-round` change.
6. Model question: config shape for two halves and per-round choice.
   Picked `review` plus new `fix`, each scalar-or-list (D9), and removal of the unused row-level model keys (D10).
7. Failure and resume questions settled as D6 and the resume acceptance criterion.

## Rejected alternatives

- **Generic two-agent producer in shuttle now.**
  Serves `discussion-advisor` too, but that note needs live dialogue and creel, a different shape; building it here designs for a requirement no task has.
- **Two parallel shed producers (reviewer row and fixer row).**
  The operator ruled it out in `two-agent-producer`: shed is linear, and fork/join in the recipe is a large change.
- **Fixer ends its turn after orienting and Go sends the review as its next turn.**
  An autonomous turn end without output files is held and sends the parent a hold notice; avoiding it needs a shuttle change for intermediate turns.
- **Fixer waits with a shell loop.**
  Not portable to Windows, and the agents' scripted-command habits are already a known problem (`agents-no-scripted-edits`).
- **Go sends the fixer the review with `Runner.Send` while it waits in a tool call.**
  `Send` requires an idle pane; the fixer is mid-turn.
- **Fixer fixes every finding with no dispute.**
  Leaves the stronger model unable to correct a cheaper reviewer's false findings, which the board entry asks for.
- **Dispute without evidence, or by severity.**
  Turns class and severity into a way to skip work, the trap `burlerengine/doc.go` names.
- **Separate keys per round (`review_round_1`, `review_round_2`, …).**
  Grows the template per round and fixes the number of distinguishable rounds in the key set.
- **Always-list `review`/`fix` values.**
  Breaks every existing scalar `review` value and needs a migration.
- **Per-round model only for the reviewer.**
  The same parser serves both keys, and a fixer that circles benefits from a change of model too.
- **Keep row-level `model`/`effort` as reviewer overrides.**
  Ambiguous between halves and unused by any row.
- **Attempt token in the marker's content instead of joining the stopped half.**
  The verb would read the file's content and compare it with an attempt it is not told, and the round would still return with a goroutine that can write; joining the goroutines before `Run` returns closes the race without a second token.
- **Mechanical enforcement of the pre-handoff read-only rule (snapshot the tree, audit write calls).**
  Today's A-before-B is prompt-enforced; adding a guard is beyond the task's goal of cost and time. Kept as an open risk.
- **Friction directive to both halves.**
  Two sessions on one note path; giving each its own note needs caller changes for no stated need.
- **Turning on a cluster fan for the review rows.**
  Not asked for; Webster-Review's forks cannot run git under the fork discipline.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Generic two-agent capability or Burler-specific? — auto-pick: Burler-specific inside `burlerengine` (D1). Resolved.
- Q2 Role names? — auto-pick: `burler-review`, `burler-fix` (D2). Resolved.
- Q3 Which half runs which gate? — auto-pick: review-parse on the reviewer, caller gates on the fixer (D3, D4). Resolved.
- Q4 Handoff mechanism? — auto-pick: Go-written marker plus `lyx burler await-review` with a cap under the Bash tool timeout (D5). Resolved.
- Q5 Failure of one half? — auto-pick: stop and remove the other half; round outcome as today (D6). Resolved.
- Q6 Result shape? — auto-pick: one outcome, per-half identities (D7). Resolved.
- Q7 May the fixer reject findings? — auto-pick: only with evidence of a false premise, in a `Disputed` section; PATTERN-review-round amended (D8). Resolved.
- Q8 Config shape for models? — auto-pick: `review` and `fix`, scalar or per-round list, last entry repeats (D9). Resolved.
- Q9 Defaults? — auto-pick: `review: sonnet[medium]`, `fix: opus[medium]`, from the operator's 2026-10-07 statements (D9). Resolved.
- Q10 Row-level `model`/`effort`? — auto-pick: remove (D10). Resolved.
- Q11 `lyx burler run` flags? — auto-pick: four per-half flags (D11). Resolved.
- Q12 Friction note writer? — auto-pick: fixer only (D4). Resolved.
- Q13 Enable a cluster fan on any row? — auto-pick: no, out of scope. Resolved.
