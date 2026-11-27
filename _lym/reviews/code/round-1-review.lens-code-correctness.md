```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function or branch the diff adds or changes on one concrete input.

**Per function or branch**
- `loomrecipe.InClosedScope`
  - darn and tapestry rows before the line, at the first Code-Review row, inside the segment, at Drift-Recheck, Describe and PR-Review, at Finalize and past it: correct.
  - Empty `currentRow` (no status file): false, or true when `roundStarted`; correct.
  - A row the recipe lacks with `roundStarted`: true; correct.
  - arras and hardener (no Code-Review segment): false; correct.
  - An unknown recipe: error; correct.
- `loomrecipe.CodeReviewRunSubdir`
  - darn, tapestry and sampler give `code`, true.
  - arras and pawl give "", false.
  - A Bouncer row with no `run_subdir` gives false.
  - All correct.
- `loomcli.scopeClosedAt`
  - Run directory absent: round 0; correct.
  - Stat fails with an error other than not-exist: the error is returned and refused with a retry; correct.
  - A pawl beam slug and an unseeded run (reads as tapestry): no error, since every seeded recipe is embedded (`internal/loomrecipe/names.go:117-126`); correct.
  - Run directory archived by a new generation or by `entryClear`, then the run sent back to the writer: reads open. **Not correct, F1.**
- `loomcli.scopeClosedRefusal`: closed, open and failed-read cases; correct. The `no row` text reaches the caller only when `roundStarted` holds without a status file; acceptable.
- Placement in `decisionVerb` and `decisionReplaceVerb`: the check runs under the decision lock, after the raddle check and before any write. A refusal releases the lock through the defer; correct.
- `shedadapters.RecordInScopeContinue`
  - Round 0 → `ErrNotEscalated`.
  - Round not escalated → `ErrNotEscalated`.
  - Cause budget → `ErrInScopeCause`.
  - An id of another class, or one the review lacks → `ErrInScopeFinding`, listing the decision ids or `none`.
  - Review missing or unparsable → wrapped error, refused with a retry.
  - A second ruling → `ErrCirclingDecided`, through O_EXCL.
  - Duplicate ids are accepted and written twice: harmless.
  - All correct.
- `shedadapters.InScopeFindings`: file absent gives nil; malformed gives an error; valid gives the ids; correct.
- `BuildRoundContext` in-scope marking: only routed, decision-class findings named in the previous round's decision file get `InScope`; correct.
- `circlingVerb` with `--in-scope`
  - At the hard cap with an escalation pending: refused before any record; correct.
  - At a Finalize halt: `continue` is refused by `finalizeCirclingVerb` (`internal/loomcli/circling.go:165`), so the flag is never silently dropped; correct.
  - `accept` never binds the flag, so it passes nil; correct.
  - The `in_scope` key appears on the envelope only when ids were given; correct.
- `circlingState.preRun` record closure: no ids → `RecordCirclingDecision`; ids → `RecordInScopeContinue` with cause decision. The cause is returned only when there is no error; correct.
- `roundCheck.priorEntry`
  - An in-scope finding resolved without a quoted title passes; correct.
  - An in-scope finding marked outdated is still refused; correct.
- `roundCheck.carriedFinding`
  - In scope with class decision: refused.
  - In scope with another class: passes.
  - Not in scope with a changed class: refused.
  - The severity and location checks still apply.
  - All correct.
- `writePriorFinding`: `InScope` takes precedence over `Held`; correct.
- `Bouncer.noticeWayForward`, closed scope: below the cap it names `--in-scope` and a board note; at the cap it names a board note and accept alone; correct.
- `Bouncer.escalationReason`, closed scope: names the board note, then the decide clause; correct.
- `Bouncer.answerRoute` with `decideVerbs`, closed scope at the hard cap: the route says to pass the id to `--in-scope` "on the continue verb below", but `decideVerbs` lists accept alone. **Not correct, F2.**
- `bouncerEntry` `scope_closed`: parsed as a bool with default false, admitted by `configRejectUnknown` and passed into `BouncerConfig.ScopeClosed`; correct.
- `gateslot` `plan_tier0_budget_sec`
  - -1 is refused; correct.
  - 0 adds no budget line; correct.
  - 45 adds the budget line after the gate lines; correct.
  - With no tiers declared the value is `none` and no budget line shows; acceptable.

**Findings**

- **F1.** MEDIUM, design. Location: `internal/loomcli/decision.go` `scopeClosedAt`, the `roundStarted` input.
  - Summary: "past the line" is read only from `round-N-review.md` files in the live Code-Review run directory. Both `archiveRunDir` paths empty that directory. So a run that has had Code-Review rounds can read as open again.
  - Concrete path:
    1. Code-Bouncer re-enters with a live settle record.
    2. `startNextGeneration` archives the directory to `code-&lt;stamp&gt;`. With `AmendAllowance == 0` (from `AmendMaxBounces`) it settles through `approveWithoutReview` and writes no round file.
    3. The run goes on to Drift-Recheck and stops there. The parent runs `goto --to Darn-Write`.
    4. `LatestRound` is 0 and the row is before the segment, so `InClosedScope` answers false, and `decision add` and `replace` are accepted again.
  - The same window opens with a non-zero allowance if the run is sent back after the archive and before the new generation's round-1 review lands.
  - The directory still holds the carried generation record, and the archived sibling still holds the rounds. Either would signal the line was crossed.
  - Evidence: `internal/shedadapters/archive.go:67-90`, `internal/shedadapters/amendment.go:41-64`, `internal/shedadapters/bouncer.go:517-531`, `internal/shedrecipe/entries_bouncer.go:356`.
- **F2.** LOW, design. Location: `internal/shedadapters/bouncerdecision.go` `answerRoute`, and its `decideVerbs` call.
  - Summary: on a closed-scope segment at or past the hard cap, the brief tells the parent to rule findings in scope with `--in-scope` "on the continue verb below". There is no continue line below, and `circlingVerb` refuses continue at the cap.
  - Evidence: the closed branch of `answerRoute` ignores the round. `decideVerbs` returns accept alone when `atHardCap(round)`. `noticeWayForward` already words the at-cap case without `--in-scope`.

Tool calls:
1. `grep` of `embeddedRecipes`, the recipes' amend settings, `circlingVerb` lines 85–130 and the `readCirclingDecision` decode: to learn whether every seeded recipe parses, whether the amendment path can settle without a round, and where the hard-cap check sits; my context lacked all of these.
2. `grep` of `finalizeCirclingVerb` and of where `AmendAllowance` is set: to learn whether `--in-scope` is dropped at a Finalize halt, and where the allowance comes from.
3. `grep` of `routed` in `roundcontext.go` and of `ParseReview`: to learn where `Held` comes from and how `RecordInScopeContinue` parses the review.
