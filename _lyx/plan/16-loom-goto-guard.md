# Card 16 — loom-goto-guard

**Edit:**
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli#loomCLI.arbiterDepsFor`
- `internal/loomcli#arbiterDeps`
- `internal/loomcli#arbiterCall.resumePrecondition`
- `internal/loomcli#loomCLI.loomConsumeAttempt`
- `internal/loomcli#arbiterFixture.deps`
- `internal/loomcli#TestArbiterVerb_VerdictOutcomes`
- `internal/loomcli#TestConsumeArbiterAttempt`
- `internal/loomcli#TestSpecFor_ScratchAndFrictionDir`
- `internal/loomcli/doc.go`

**Create:**
- `internal/loomcli#settledSegmentGuard`
- `internal/loomcli#TestSettledSegmentGuard`

**Uses:**
- `internal/shedadapters#SettleLive`
- `internal/shedengine#GotoGuard`

**Intent:** D2 for loom and darn: a goto never re-runs a settled review, bar the verify retry.

`settledSegmentGuard` answers, for a target row with a non-empty segment whose segment holds a Bouncer row (`loomrecipe.BouncerRunSubdir`), and which is a review or fix row of that segment, not the Bouncer itself:
when `shedadapters.SettleLive` over the segment's run directory reports a live record (settled or interrupted alike), it refuses with `the <segment> verdict is settled; way forward: "lyx loom goto --to <Bouncer>" re-settles it at no review cost, then "lyx loom resume" in the task worktree runs it, or "lyx loom goto --to <writer>" re-runs the writer, and a changed subject then starts a new generation`, where the writer is the first row in list order outside the segment whose `OnDone` names the Bouncer, so Plan-Review names `Plan-Write`, never `PR-Rework`;
the one admission over a live record is a Publish failure record whose `Target` names the target row and whose `WrittenAt` is later than the record's settle time, loom's verify retry onto `Webster-Burler-Review`; a record with no `WrittenAt`, as an older binary writes it, admits the goto as today.
Every other target (the Bouncer, a row outside every review segment, a segment whose record is not live, an amendment under way included) is admitted.
A read error of the record or the failure record returns the error, which Shed turns into a refusal with a re-run way forward.
Bound: the guard only refuses, and the bypass covers only the row the failure record names until that generation settles again, after which the new settle time is later than the write time and the goto is refused for every caller; a case 1 re-settle keeps the stored settle time, so a goto onto the Bouncer after a Publish failure leaves the bypass open.

Wiring: `loomCLI.specFor` sets `Spec.GotoGuard` when the location is known (so `lyx loom goto`, `lyx shed goto` on a loom run, batten's automatic `lyx loom goto` and the Go driver's verify goto all pass it); `goDriverDepsFor` sets `TacklerDeps.GotoGuard` and `Guard` on the tackler consume's `RelaunchRequest`; `arbiterDeps` gains `GotoGuard`, filled by `arbiterDepsFor` and run by `arbiterCall.resumePrecondition` after `CheckGotoTarget`, so a verdict naming a refused row is rejected as invalid; `loomConsumeAttempt` sets `Guard` on its `RelaunchRequest`, so a refusal inside the arbiter's re-launch refuses the attempt and the driver hands back with the refusal text (card 6 pins the driver side).
Batten's own consume in `internal/battencli` tells no guard and stays as it is, since batten's recipe has no review segment.
`doc.go`'s `# Arbiter` paragraph adds, to its list of the preconditions that make a `resume` unsound, a goto target the told goto guard refuses;
the paragraphs on the tackler's and the arbiter's re-launch say that a guard refusal inside the re-launch refuses the attempt and the driver hands back with the refusal text.

`TestSettledSegmentGuard` is one table over a loom producer list and run directories holding records: the Bouncer row is admitted under a live record; a Plan-Review review row and a fix row are refused, the text naming `Plan-Bouncer` and `Plan-Write`; a Merge-Review review row is refused, the text naming `Merge-Bouncer` and `Merge-In`; an interrupted (unsettled) live record refuses the same; a record a later round's report made not live admits; an amendment generation that has not settled admits; a failure record naming `Webster-Burler-Review` written after the settle admits that row and refuses the fix row; one written before the settle refuses; one with no write time admits; a record unreadable as JSON returns an error; a failure record that does not decode returns an error.
`TestConsumeArbiterAttempt` adds a row whose base `RelaunchRequest` carries a refusing `Guard`: the consume returns the refusal, writes no applied mark and leaves the status unchanged.
`TestSpecFor_ScratchAndFrictionDir` asserts `specFor` sets a non-nil `GotoGuard` when the location is known, so every goto through loom's spec passes the guard.
`TestArbiterVerb_VerdictOutcomes` adds a row whose fixture guard refuses the verdict's goto: the verdict is handed on as invalid naming the refusal; `arbiterFixture.deps` carries the guard.

**ImpactSummary:** loom's goto, the arbiter's and tackler's verdict checks and both re-launch consumes gain a refusal over settled review segments; batten's own runs tell no guard.

**Commit:** 16: loom: refuse a goto that would re-run a settled review, bar the verify retry
