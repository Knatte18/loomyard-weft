# Webster and plan review honour parent decisions; full halt findings

Parent decisions now reach the checks that used to ignore them, and Webster halts name the files and routes a parent needs.
Issues #168, #170, #171, #172 and #178.

## What changes

**A decision can drop a Create (#171, #178).**
`lym loom decision add --drops-create <target>` (repeatable, one entry per call) writes a ``Drops Create: `<target>` `` line into the added entry.
`discussionparser.DroppedCreates` reads those lines.
`runplan.PostBatchChecks` takes the decision record through the new `PostBatchInputs.DecisionRecordPath` field.
It runs the done-checks on a copy of the batch's cards without the dropped Create refs, and returns a warning per skipped target, so the batch no longer blocks on a member the parent decided never to build.
Webster and weaver both pass the path.
`createTargetWayForward` no longer depends on the calling verb:
record-batch and recover-batch both offer, in order, the card amendment plus `lym webster rebaseline --card NN`, the `--drops-create` decision, and recover-batch to build the target as the card names it.
A signature mismatch still offers the amendment alone.

**Verify-gate halts name the full report (#172).**
A run-level entry's stuck reason still quotes the first 400 bytes of its findings, then names `verify-gate.yaml`, which holds them all, and marks a cut.
A failure of the `drift` entry closes its way forward with the parent's `lym drift amend <slug> --entries <file>` route for a drift the plan intends.

**Plan review admits amended cards (#168).**
`burlerengine.RoundContext` gains `AmendingTexts`: the added entries whose titles the plan amendment record lists as pending.
In an incremental round, a finding on a card whose file stem, or stem without its number, appears in such an entry passes the changed-file rule; it must still be BLOCKING.
The prior-findings block marks amending entries and tells the reviewer to raise a finding on each named card that does not yet carry the amendment.

**Webster's halt reason reaches the plan fixer (#170).**
`WebsterProducer` writes its final stuck reason to `.lym/implementer/webster-halt.md` (`runplan.WriteHaltReason`) and removes it when a call ends done.
Plan-Burler-Review reads it on every round into `Profile.HaltReason` and `RoundContext.HaltReason`.
The review prompt then asks for a BLOCKING finding with `origin: webster` (`burlerengine.WebsterOrigin`), which the round check admits outside the changed files, so the fixer works it.
Only the plan segment is wired: `shedrecipe` sets the amendment reader and conflict directory for `Plan-Burler-Review` alone, and `lym burler validate-review` does the same for the plan review run directory.

**A later parent decision wins (#178).**
The master prompt numbers recorded parent decisions `Decision 1:` onward in record order.
The master and implementer stencils state that a later decision that differs from an earlier one supersedes it.

## Design decisions

- A dropped Create is named by an explicit flag, not inferred from decision prose, so a decision that merely mentions a target never unblocks its done-check.
- `shedadapters` cannot import `loomshed`, which imports it, so the amendment titles reach `BuildRoundContext` as a reader func that the callers build on `loomshed.ReadAmendment`. That keeps the record to one decoder.
- `BuildRoundContext` takes a `RoundContextSources` struct in place of its two trailing path arguments.
- Left out per the parent's scope decision: running `lym webster rebaseline --card N` automatically after a plan fix (#170, second bullet), and stopping the verify-gate re-prompt loop early on unchanged drift findings (#172, a "consider" item).
- #170's suggestion to refuse a Plan-Bouncer approval while the halt finding stands is not built. The halt reaches the fixer as a finding, and the round checks keep that finding open across rounds like any other.

## Merge danger

**Door:** one-way
**Blast radius:** webster

The `Drops Create:` line is a new line kind in the decision record, which other lym binaries read; an older binary treats it as entry text.
