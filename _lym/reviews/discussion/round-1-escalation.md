---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `board-viewer-build` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the soft-cap rule could not read its inputs: the round's review, its ledger or the previous round's ledger.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.
- `stalemate`: the review target stayed unchanged for the configured number of rounds in a row.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Put each remaining finding to the operator, unless a recorded operator decision answers it.

- `F9` (MEDIUM): The entry's Delivers line names `lym board open` with 'its refusal naming lym board rerender', and D5 drops it. Whether the committed `board.html` and `board-data.js` still earn their place now that editing needs the server is a question about what the task is for, and no Added after Discussion entry or `Decision for board-viewer-build:` note answers it. Fix: Raise both questions in the Open risks line to the run's parent: confirm the dropped refusal, and say whether the two committed render outputs stay. Record the answer with `lym loom decision add`.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D5 resolves `lym board open` with `lymcwd.ResolveWithin`, which PATTERN-cwd-resolution reserves for a read-only verb; `open` starts a board-writing server and writes a launcher and state, so it is no read-only verb. Fix: In D5's first bullet, resolve `open` with `lymcwd.Resolve`, as the other board verbs do; the VS Code task's cwd is the workspace folder, which is the anchored directory. Drop 'so the verb runs from anywhere inside the worktree's anchored directory or its records sibling'. If the operator still wants `open` to run from any subdirectory, add a `## PATTERN changes` bullet `PATTERN-cwd-resolution — changes:` that lets a verb that starts a hub server call `ResolveWithin`, and state that bound in D5.
- `F2` (BLOCKING): D9 and D11 rest the token's protection on file mode 0600, but Windows, which Scope In covers, ignores Go's permission bits, so on Windows another user can read the state file or the launcher and write the board as the operator. Fix: In D9's second and last bullets, state that the state file and the launcher are written 0600 and, on Windows, with an owner-only DACL, as `internal/githubclient/cache_windows.go` already does. Cite `internal/githubclient/doc.go` as the pin for 'Windows ignores Go's permission bits'. Extend the D9 criterion so that on Windows the file's DACL grants only the owner.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, and at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add board-viewer-build`, then record the segment's outcome with exactly one of:

- `lym loom circling continue board-viewer-build` for a round over the amended record
- `lym loom circling accept board-viewer-build` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/board-viewer-build/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/board-viewer-build/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/board-viewer-build/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/board-viewer-build/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/board-viewer-build`.

## What to do

Settle the question only when a recorded operator decision answers the envelope's question directly.
A recorded operator decision is one of:

- A passage of the task's board entry that the entry marks as an operator decision with a date, as `Operator decision (<date>)` does.
- A board note whose body states that it records an operator decision.
- A decision-record entry added with `--by operator` that is already in the committed decision record when the escalation arrives.
- A Discussion answer that the support log's `## Interview` shows the operator gave.

These never count: an `Idea` or agent-suggestion passage, an operator view not marked as a decision, a decision-record entry added with `--by arbiter` or `--by parent`, a writer's auto-pick, and an `## Auto-mode assumptions` entry.
`--by` is an unverified label, since only `--by arbiter` is restricted, so a `--by operator` entry counts only when it was in the committed decision record before the escalation.
Never settle a question whose answer widens the task's scope, and never accept over an open design-wrong finding unless a recorded operator decision covers that finding.
At cause `budget`, decide nothing: report back to your parent session which input could not be read.

When a recorded operator decision answers the question:

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add board-viewer-build --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/board-viewer-build`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept board-viewer-build` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue board-viewer-build` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/board-viewer-build`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
