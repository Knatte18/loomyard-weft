---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-pr-align-producer.md:46
    summary: PR-Align step 7 reads the decision snapshot before Begin but states no outcome for a Snapshot error, so a board or decision-record read failure at spawn becomes a returned producer error instead of today's path, unlike the parent-review side and with no test row
    fix: In card 10's Intent step 7, state that a `Snapshot()` error is handled like a `Begin` error, with nothing spawned, a note written with the error as its reason, and `Done`. Add a `TestPRAlign` row reading "A Snapshot that fails spawns nothing, writes the note naming the error and ends Done", driven by a `Deps.Alignment.Snapshot` closure that returns an error.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-parent-review-gate-alignment.md:56
    summary: Cards 7 and 10 give no outcome for an error from Aligner.Poll / Session.Poll or from Close, which card 6 returns on a probe failure after a restart, an unreadable attempt.json or a failed StopStrand, so these failures become a gate or producer error that fails the run instead of falling back
    fix: "In card 7, step 3, state that an `Aligner.Poll` error is treated as a done poll with the error text as its reason, which means close with `fallback`, rewrite the brief through `RenderAlignmentBrief` and continue into delivery. State that an `Aligner.Close` error during a fallback, a parent verdict, expiry or `final` is logged, and the record is still settled through `SettleAlignment`, so it is never polled again. Add `TestGate_Alignment` rows: (a) a Poll that errors gives no gate error, writes a brief naming the error and carries the delivery prompt; (b) a Close that errors on fallback still settles the record and delivers. In card 10, step 8, state that a `Poll` error closes the attempt, writes a note naming the error and is `Done`. Also state that a `Close` error is logged and the note is still written. Add a `TestPRAlign` row: a Poll that errors writes the note naming the error and ends Done."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/10-pr-align-producer.md:57
    summary: Card 10 never says which pull request number and head a note records when the head or number moved between spawn and verdict, but PR-Gate shows a note only when it matches the current pull request, so a note keyed to the spawn head drops the alignment reason that D5 says PR-Gate's awaiting notice must name
    fix: In card 10's step 9, state that every `AlignmentNote` records the pull request number and head read at apply time, which are the re-read values, and not the spawn's. For the Begin-error and pause notes, which have no re-read, it records the values read at step 4. Extend the `TestPRAlign` row "A head that moved during the wait writes no record" so it also asserts that the note carries the new head. That way, `TestPRGate_Verdicts`' matching-note row covers what PR-Gate then reads.
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-alignment-session.md:57
    summary: Several guard branches that cards 6 and 10 prescribe have no named test row that reaches them
    fix: "Card 6 `TestSession` gains these rows. (a) A failed strand record: `fakeShuttle`'s Start hook replaces the attempt directory with a plain file, so recording `StrandGUID` fails. Assert that `Begin` returns the error and the strand is stopped once. (b) A run released with a shuttle timeout result gives `CauseTimeout` through the watch, not through the deadline. (c) A run whose `Wait` returns an error gives `CauseDied`. (d) A second `Close` on a closed attempt stops nothing and leaves `ClosedReason` unchanged. Card 10 `TestPRAlign` gains these rows. (e) A non-transient GitHub failure, a 404 or 422 answer from the server, ends `Done` and spawns nothing. (f) A matching rejection record spawns nothing. (g) An open attempt with the key `pr-<number>-<head>` is passed to `Begin`, and no new spawn is counted: this is the restart-reuse criterion. (h) A `Config` closure that errors returns the error and spawns nothing."
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/10-pr-align-producer.md:65
    summary: Card 10 applies an agent reject through RecordRejection before MarkApplied records it, so a restart between the two leaves the attempt with no applied reject while step 5's matching-rejection exit ends Done, and a later agent reject then counts as the first, breaking D5's once-per-run reject count
    fix: In card 10's step 9 reject branch, call `MarkApplied(reject)` before `RecordRejection`. If `RecordRejection` then fails, a later agent reject only escalates, which is the safe direction. Extend the `TestPRAlign` row "A first reject writes a rejection whose findings are the report" so it asserts that the applied mark is recorded before the rejection file appears, using the fake session's recorded order.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/12-orch-stencil-alignment.md:30
    summary: Card 12 keeps every existing orch claim, but the claim at contracts/stencils/claims_test.go:691 forbids `alignment-agent` because overlap checking stays with that board note, which is false once this task lands
    fix: In card 12, under the `wordingClaims` bullet, add a line saying that the `orch-template-role.md` claim forbidding `alignment-agent` (contracts/stencils/claims_test.go:691) keeps its mustNot and has its why rewritten to say that the stencil names the alignment agent in prose and never by its board-note slug.
---

### [MEDIUM] PR-Align's snapshot read at spawn has no stated failure outcome

**Location:** _lyx/plan/10-pr-align-producer.md:46

**Issue:** Step 7 reads `Alignment.Snapshot()` and then calls `Begin`, and only a `Begin` error is given an outcome: a note is written and the row ends `Done`.
The production snapshot from card 8 reads the decision record and the board through `hubBoardTasks`, so a board read failure is a realistic input.
Card 10 gives that error no outcome, so it falls out of `Call` as a returned error.
D5 says `PR-Align` ends `Done` except on an error a producer returns today, and every failure ends in today's path.
On the parent-review side, card 8's `Spawn` reads the snapshot inside `Spawn`, so the same failure becomes an `Aligner.Spawn` error, which card 7 sends to delivery.
The two points handle the same spawn-time failure in opposite ways, and no `TestPRAlign` row reaches it.
This is focus directive 1: card 7's spawn-error path does end in delivery, with the row "A `Spawn` that returns an error gives no gate error", and card 10's `Begin` error does end in `PR-Gate`, with the row "A `Begin` that fails". The snapshot read that comes first in card 10 is the uncovered part of the start sequence.

### [MEDIUM] Poll and Close errors have no outcome at either alignment point

**Location:** _lyx/plan/07-parent-review-gate-alignment.md:56

**Issue:** Card 6 has `Session.Poll` and `Session.Close` return errors.
`Poll` re-reads `attempt.json`, probes shuttle after a restart through `ProbeGated`, and calls `StopStrand` on a timeout.
`ProbeGated` can refuse on reed gates and multiplicity.
`Close` returns a `StopStrand` failure and leaves the attempt open, which `TestSession` pins.
Card 7's `pollAlignment` and `closeAlignment` return `error`, and the gate's existing convention (`internal/parentreview/gate.go`) returns every closure error as a gate error.
Card 7 never says that these errors are caught.
A probe failure after a driver restart would therefore fail Discussion-Write's gate outright, where D4 wants a dead or unreachable agent to fall back to today's delivery ("every failure ends in today's path").
Card 10 also calls `Poll` in step 8 and `Close` on every path without saying what an error does.
Neither card has a test row for these errors.
Once the gate's record is no longer `running`, it is never polled again (card 7, step 4).
So settling the record even when `Close` fails still meets D4's guarantee that a verdict written after a fallback is never applied.

### [MEDIUM] An alignment note's pull request number and head are unspecified when the head moved

**Location:** _lyx/plan/10-pr-align-producer.md:57

**Issue:** `AlignmentNote` carries `PRNumber` and `HeadSHA`.
`PR-Gate` names a note only when it "finds a note matching the pull request number and head", and a `TestPRGate_Verdicts` row pins that a note for another head adds nothing.
Step 9's moved-head branch "writes a note saying so" but does not say whether it records the spawn's number and head or the re-read ones.
If it records the spawn's, the next `PR-Gate` sees the new head and finds no matching note.
Its awaiting notice then omits the alignment reason, against D5's "a head or pull request number that moved since the spawn … records the report path or the reason. `PR-Gate`'s awaiting `ParentNotice` names it".

### [MEDIUM] Guard branches in the session and PR-Align cards with no test row

**Location:** _lyx/plan/06-alignment-session.md:57

**Issue:** This is focus directive 3.
Card 6 prescribes four branches that no `TestSession` row reaches:
- "A failed strand record stops the strand before returning" (line 57).
- `Poll`'s watch-ended `CauseTimeout` for a shuttle timeout result and `CauseDied` for a wait error (line 62). The deadline row reaches a different branch, line 63.
- "Closing an attempt that is already closed changes nothing" (line 68), which card 7's `final` and parent-verdict paths rely on.

Card 10 prescribes four branches that no `TestPRAlign` row reaches:
- Step 4's "any other failure … is `Done`, with a log line", for a non-transient GitHub failure or an unusable origin or client.
- Step 5's matching rejection.
- Step 6's reuse of an open attempt through `Begin` after a restart. This is the only PR-side row for the criterion "A driver restart during `PR-Align` reuses the live spawn … and spawns no second agent".
- Step 3's returned `Config` error.

Each named fix row gives a concrete input that reaches its branch.

### [LOW] An agent reject is recorded before it is counted

**Location:** _lyx/plan/10-pr-align-producer.md:65

**Issue:** The reject branch writes the rejection record and only then calls `MarkApplied(reject)`.
After a restart between the two, step 5's "an approval or rejection record already matches" exit ends the row `Done` before step 6 or step 9 runs again.
The attempt keeps an empty `Applied` for good.
After `PR-Rework`, a second agent reject finds no earlier attempt whose `Applied` is `reject`, and it is applied.
That breaks D5's "allowed once per run, counted in a durable attempt record … that survives restarts".

### [LOW] An orch claim's reason goes stale when the alignment agent lands

**Location:** _lyx/plan/12-orch-stencil-alignment.md:30

**Issue:** The `orch-template-role.md` row in `wordingClaims` carries `{mustNot: "alignment-agent", why: "checking the run against other runs in flight stays with that board note, so the stencil does not ask for it"}` (contracts/stencils/claims_test.go:691).
Card 12 makes the stencil describe that check as the alignment agent's and says "the existing claims keep passing".
The claim still passes, since card 12's wording is "alignment agent" and never "alignment-agent".
Its stated reason, however, contradicts the stencil it guards after this task.

## Focus verdicts

- Directive 1, spawn failure: card 7 sends an `Aligner.Spawn` error to today's delivery and has a row that reaches the branch. Card 10 does the same for a `Begin` error and has a row. The gaps are the PR-side snapshot read (F1) and errors raised after the spawn (F2).
- Directive 2, the fallback notice cut: not a defect. `PR-Gate`'s reason reaches the batten status error verbatim through `stuckReason` (`internal/shedengine/run.go:36`). `renderNotice` makes paths under the pair relative before it cuts at `noticeErrorMax` (200 runes, `internal/battenshed/notice.go:150`). Card 10 puts the clause, report path first, at the head of the reason. The relative report path is `_lyx/shed/<slug>/alignment/pr-gate/attempt-<N>/report-<16 hex>.md`, about 65 runes plus the slug. With the leading "alignment report " it stays well inside 200 runes for any realistic slug. The pull request URL and the existing reason that follow it may be cut, but the path survives.
- Directive 3, guard reachability: see F4.
