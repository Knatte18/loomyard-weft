# Support log — orch-intake-fork

## Interview

### Turn 1 — exploration

Autonomous run, no advisors.
Read the board entry, `orch-split` and `labels-from-diff`, the intake CLI and engine, the orch role stencil and the `ly:board` skill.
`intake list` and `import` both print issue bodies; nothing else in Go or `tools/` parses their output.
The import payload already takes `labels` for a new note, so the fork can label in one call.

### Turn 2 — self-picks

Settled Q1–Q6 in the question ledger as auto-picks and wrote the record.

### Turn 3 — lens review

Six forks reported; findings merged and judged:

- pattern-fit MEDIUM, D4 put command mechanics in the policy-only stencil: fixed, the stencil states outcomes and bounds and points to `ly:board`.
- premises MEDIUM, the open risk named the wrong lag (stencils are seeded by the binary): fixed, the risk is the rendered role file and loaded skill until the next cycle.
- scenarios MEDIUM, a failed close after import is orphaned: fixed, D4 has the fork re-run the `close` payload.
- scenarios MEDIUM, no failed outcome in the return line: fixed, D4 adds `failed` with the error.
- scenarios MEDIUM, no rule for a `list` error: fixed, D3 treats it as a lyx bug and starts no fork.
- scenarios LOW and goal-scope LOW, two live forks after a cycle: fixed, D3 states why the CLI settles each issue once; a noise close beating an import is the fork's own judgment, not loss.
- scenarios LOW, a label upsert may drop labels: fixed, D5's skill states the upsert carries the whole label list.
- scenarios LOW, `[]` for `--full`: fixed in acceptance criteria.
- goal-scope LOW, label write on a folded entry unjustified: fixed, D4 cites the board policy.
- goal-scope LOW, D5 beyond the entry's Idea list: fixed, D5's rationale says D1 and D2 make the skill wrong.
- pattern-fit LOW, stencil criterion should be by review: fixed.
- pattern-fit LOW, no-denied-recovery should name both files: fixed.
- leanness LOW ×6 (github-auth constraint, duplicate docs constraint, generic test-economy line and last criterion, auto-mode restating decisions, `intakeIssueView` note, D1 precedent sentence): fixed by cutting or shortening; test-economy kept, reworded to the prose-pinning rule pattern-fit raised.
- premises LOW, a fold-label upsert cannot hit the run lock: rejected, the plan touches only prose there and adds no refusal handling.
- premises LOW, D2 removes fields only: rejected, a confirmation needing no change.
- goal-scope LOW, `--full` absent from the command index: rejected, informational; D4 and D5 name the flag where the fork needs it.
- board-fit LOW ×4 (`orch-split`, `labels-from-diff`, `stencil-intent-audit`, compact stencil): rejected, informational with no conflict.

## Rejected alternatives

- A `--compact` flag keeping full output as the default: the orch would pay for every run that forgets the flag.
- An `intake show {issue}` verb for bodies: `--full` serves the fork with one call.
- Tracking an open intake fork in the orch note: the open issues themselves are the state, and a duplicate import is a no-op.
- A fresh agent instead of a fork: the operator chose the fork for its inherited board policy.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): compact by default, or a compact flag? Picked compact by default; no Go caller parses the output, and the cheap form should not need remembering.
- Q2 (auto-pick): name of the full-view flag? Picked `--full`.
- Q3 (auto-pick): import output fields? Picked slug, issue, labels, dropped, the same for a fold, and the same fields in the close-failure envelope.
- Q4 (auto-pick): when does the orch check? Picked every wake, the landed notice included, as today.
- Q5 (auto-pick): what may the fork write? Picked intake verbs and label upserts only, with fold and close bounded by the board policy.
- Q6 (auto-pick): update the `ly:board` skill? Picked yes, since the fork follows it.
