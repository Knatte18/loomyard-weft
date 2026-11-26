---
round: 1
cause: decision
---
# Escalation — review segment `Discussion-Bouncer`, round 1

You are a one-shot fork of the parent session of a tapestry run.
The run's review segment `Discussion-Bouncer` for task `fabric-git-free-core` has stopped at round 1 and is `awaiting` a decision.
The cause is `decision`:

- `circling`: the judge found no progress on recurring gating findings, or a design-wrong finding returned after a continue.
- `budget`: the budget is spent and the round cannot settle as an accept by itself:
  BLOCKING findings are still open, the round's live gate is red, or the soft-cap rule could not read its inputs.
  The detail below, when present, names which.
  A round whose review also holds `decision`-class findings reaches this cause when such BLOCKING findings are open,
  and its detail names those findings.
- `hard-cap`: the round is at or past the perch's hard cap with a design-wrong finding still open, or a continue was recorded there.
- `decision`: the round's review holds a finding the review cannot settle itself.
- `stalemate`: the review target stayed unchanged for the configured number of rounds in a row.

### Decision findings

Each finding below is a question the review could not settle itself.
Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it.
Settle each remaining finding yourself when it is a run-level call, and put it to the operator only when it is a design question, unless a recorded operator decision answers it.

- `F18` (MEDIUM): Open risk 1 leaves to the parent whether push-retry-not-halt's dependency on this slug should point at the push and origin part or at the last part of the chain. It is a board edge on another entry, so only the parent can set it. Fix: Parent: decide which part of this chain push-retry-not-halt depends on (part 2, push and origin, or the close-out part 7), and record it with lym loom decision add so raddle and the board edge follow. Once it is answered, the writer drops Open risk 1 and its Q12 ledger line, and Open risk 2 with it if the answer settles the order of hardener-study-faults and keep-landed-run-records.
- `F19` (MEDIUM): Open risk 3 asks whether the operator's 'fabric's tests test fabric alone' also asks for a slimmer internal/hubforge. hubforge imports fabriccli and hubreconcile, so fabric's tests stay in the gate scope of modules above fabric, and every other package whose tests use hubforge stays in fabricengine's scope (F9). The answer bears on the operator's intent and the task's scope. Fix: Parent: decide whether this task leaves hubforge's import set as Scope says (Out, hubforge's template machinery and its import set), or takes in a hubforge split so that fabric's tests and name-only importers stop reaching modules above fabric. Record the answer with lym loom decision add, after which the writer drops Open risk 3 and its Q15 ledger line.

### BLOCKING findings of the round's review

- `F1` (BLOCKING): D13 and the declared PATTERN-fabric-boundary text count internal/fabriccli as fabric and bar fabric from importing any module above it, but fabriccli imports pairteardown, boardengine, hubreconcile, shedengine, shedrun and other engines. It also hosts both detached children's entry points, so no decision says where their seams are built, and D13's 'package that imports internal/fabricengine' clause can never fire on fabricengine, fabriccore or fabriclayout. Fix: In D13, define 'above fabric' as a named set or rule that does not depend on importing internal/fabricengine: the four cut modules and every package that imports one of them or fabricengine, for example. Say whether internal/fabriccli is bound by the cut, and decide where the teardown child (internal/fabriccli/teardownchild.go) and the pair push child (internal/fabriccli/weft_verbs.go, fabricengine.DetachedPush) get their run-path, run-lock, status and takeover seams. Then make the PATTERN-fabric-boundary changes bullet and the D13 acceptance criterion match that decision. Either name fabriccli's own cuts and add a D9 part for them, or keep fabriccli out of the 'imports no module above it' clause and out of the guard.
- `F2` (BLOCKING): D13 treats shed's data as facts a read seam hands fabric, but fabric writes shed's records itself (codetipsettle rewinds and commits a shedengine.Status under shedrun.StatusLock; takeoverwrite writes, commits, pushes and rewrites a shedengine.Takeover). Public fabric readers such as OriginRunStatus, OriginParentRunStatus, OriginBattenStatus and ReadUpstreamTakeover also return shed types from git reads only fabric may run, and a seam of flags, refs, paths, lock handles and verdicts cannot carry those record shapes. Fix: In D13's shedengine/shedrun bullet, decide for the record writes (codetipsettle.go, takeoverwrite.go) and the readers (originreads.go, handover.go, takeover.go) how they cross the cut, and state D5's guarantee for each. The options are: shed supplies an encoder and decoder through the seam, fabric carries opaque bytes at a path the seam names, or the read-and-write moves out of fabric into a shed-side caller of fabric's general git verbs. Add an acceptance criterion that a takeover write and a code-tip settle produce the same status file bytes as before.
- `F3` (BLOCKING): D13 has the IDE module pass the hub workspace in as bytes, but fabric regenerates the workspace inside its own verbs after their state changes: every Add exit, rollback included, Remove's deferred regeneration and the anchor change. No caller holds those bytes before the verb runs, so the cut as written cannot keep D5's behaviour. Fix: Rewrite D13's vscode bullet: the caller supplies a workspace builder seam, which fabric calls with the folder list in fabric's own vocabulary after the verb's changes, and which returns the bytes fabric writes through the hub os.Root. State the seam's bound, including what fabric writes when the builder is unset or fails. Name the existing installed hooks (hubworkspace.go InstallWorkspaceContent, InstallPairFolderNamer) as what the seam replaces or keeps.

### Refusal

none

When a refusal is named, the round is re-judged before the recorded answer acts while budget is left.
An `accept` passes the round over every BLOCKING finding listed once a judgment is admitted, except that over a CONTINUE or CIRCLING verdict an open BLOCKING ledger entry of another class holds it: the round then runs another round, or inside the soft-cap window escalates again with cause budget, which only the parent accepts. An `accept` at or past the hard cap halts the run if the refusal still holds at the budget.
A `continue` runs another round whose fresh reviewers re-review the artifact, so a listed BLOCKING finding reaches that round's fixer only if they raise it again.

### Way forward

Record the answer with `lym loom decision add fabric-git-free-core`, then record the segment's outcome with exactly one of:

- `lym loom circling continue fabric-git-free-core` for a round over the amended record
- `lym loom circling accept fabric-git-free-core` to proceed

Then resume the run with `lym loom resume`.

What a ledger entry's `design-wrong` and `mechanical` tags mean is defined in `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/bouncer/bouncer-template-judge.md`.

## Materials

Read these before you decide anything:

1. The arbiter attempt's `envelope.json`, when your parent session names one: read it first, since the arbiter already judged the review materials by its own rule and its `question` is what you answer.
2. The round's review at `/home/knatte/Code/loomyard-LYMHUB/fabric-git-free-core/_lym/reviews/discussion/round-1-review.md`.
3. The round's ledger at `/home/knatte/Code/loomyard-LYMHUB/fabric-git-free-core/_lym/reviews/discussion/round-1-bouncer-ledger.md`.
4. The judge's verdict at `/home/knatte/Code/loomyard-LYMHUB/fabric-git-free-core/_lym/reviews/discussion/round-1-bouncer-verdict.md`.
5. The decision record at `/home/knatte/Code/loomyard-LYMHUB/fabric-git-free-core/_lym/discussion/decision-record.md`, when it exists.
6. The support log `support-log.md` in the decision record's directory, read when the definition below needs its `## Interview`.

The run's worktree is `/home/knatte/Code/loomyard-LYMHUB/fabric-git-free-core`.

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
At cause `budget`, decide nothing: report back to your parent session what the cause names,
which the parent settles by accepting the round or ordering another one.

When a recorded operator decision answers the question:

1. Record the operator decision you applied, citing where it is recorded, with `lym loom decision add fabric-git-free-core --by parent --title … --decision … --rationale …`, run in `/home/knatte/Code/loomyard-LYMHUB/fabric-git-free-core`, when the decision record exists; never record with `--by operator`.
   Without a decision record, name the applied decision and its source in your report back to your parent session instead.
2. On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept fabric-git-free-core` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue fabric-git-free-core` runs one more round.
3. Resume the run with `lym loom resume` in `/home/knatte/Code/loomyard-LYMHUB/fabric-git-free-core`.

Otherwise record nothing and report back to your parent session, which asks the operator.
The run stays `awaiting`.
