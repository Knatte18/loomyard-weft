# Fixer report — discussion, round 1

Every finding's premise held against the code at HEAD; all sixteen are fixed.

## Changes

- F1 (fixed as proposed): D12 fixes the attempt's strand identity before the spawn, its report path unique to batch and attempt number, records it with attempt and rung, and attaches by that identity through shuttle's output-file attach (`AttachGated`, `internal/shuttleengine/attach.go`); the `FindRun` claim is gone. The D12 criterion keeps never-spawns-twice.
- F2 (fixed as proposed): D3 records the full partition at each batch begin, done batches kept and new batches numbered after them; D11 says the shared checks read that recorded partition. New criterion: a batch removing a symbol a pending card names is refused by the later-reference check, and only the last batch renders as last.
- F3 (fixed as proposed): D13's seam picks the implementer whose run directory holds begun batches, falls back to the setting, and refuses on both-begun or an unloadable state file; D1's guard reads through it, a missing file counting as none. The D13 criterion covers the flip to `webster`.
- F4 (fixed as proposed): D13's renumber bullet takes the active implementer's state-mutation lease around the bound check and renumber; D3 records the batch begin under that lease. The D13 criterion names the race.
- F5 (fixed differently, after the self-check): the proposal to drop the hold-back was first carried out, then reversed. The fix-proposal lens showed a halted run's leftover strand, woken by the notice, would edit the tree beside the arbiter, the race D12's rationale names. Instead the hold-back stays and D12's attach sends the strand every decision entry recorded since its attempt began, so a decision recorded during the halt reaches it. Criteria in D12 and D13 match.
- F6 (fixed as proposed): D13's reader list adds Describe's live run record, its prior-generation readers and the PR rework archive seam, which archives `_lym/weaver/` into `prior-generation/weaver`; D9 and D10 point at D13. New criterion names these readers and merge-in.
- F7 (fixed as proposed): D11 settles it from `postBatchInputs`, `checkRecoveredSuspects`, `failureDigestBlock` and the fingerprint helpers' signatures: the run-plan state types move into the shared package and both state files embed them; `Geometry`'s run directory names the active implementer's. D9 states Weaver's state embeds the shared state and adds its attempt fields.
- F8 (fixed as proposed): D6 stops a failed or timed-out attempt's strand through shuttle's stop verb and records it failed before the next attempt or the Stuck; D12 attaches only to an attempt recorded in flight.
- F9 (fixed as proposed, extended after the self-check): D1 names `lym webster reset --to start` and `lym weaver reset` by direction, and a test reaches each. D2's Bound adds `lym weaver reset`, which moves the branch back to the run's start commit and archives `_lym/weaver/`, refusing while Weaver's run lock is held. Scope In names the verb.
- F10 (fixed as proposed): D2's Bound names `tackler_max_spawns`, `arbiter_max_spawns` and the unbounded parent re-step; D5 points at D2; the repair cap is gone.
- F11 (fixed as proposed): D10 gives the fixer `gatefixer`'s verify-fixer role and `weaver.yaml`'s `fixer_timeout_min`, default 60, which batten's watch reads for a Weaver run; D5 names the key.
- F12 (fixed as proposed): D13's list adds merge-in's in-flight warning.
- F13 (fixed as proposed): D8 keys the append by its batch-and-attempt heading and skips it when present.
- F14 (fixed as proposed): D6 gives each attempt, re-entry and fixer strands included, one same-rung retry; D5's Bound says so.
- F15 (fixed as proposed): the Scope Out line on pushing is dropped.
- F16 (fixed as proposed): support-log Q6, Q9 and Q10 are marked revised, with D7's, D10's and D8's answers.

## Self-check

- fix-consistency, LOW, the Turn 4 log line still said the attach goes through `FindRun`: fixed as proposed, the line notes the round-1 revision.
- fix-claims, MEDIUM, D12 said `shuttleengine` gains an output-file lookup that `Attach`/`AttachGated` already provide: fixed as proposed, D12 names `AttachGated` and counts a respawn-eligible run the attach clears as a machinery loss.
- fix-claims, LOW, D11 said every fingerprint helper takes `*State`: fixed as proposed, narrowed to the helpers that compare or restamp.
- fix-way-forward, MEDIUM, a corrupt state file's refusal named a reset that cannot load it: fixed as proposed, D1 names repairing the file or moving it aside.
- fix-way-forward, MEDIUM, `lym webster reset` named with no target: fixed as proposed, `--to start`.
- fix-way-forward MEDIUM 3 and fix-proposal MEDIUM 2 (merged), `lym weaver reset` left Weaver's commits on the branch: fixed as proposed, the verb moves the branch back to the run's start commit, as `--to start` does.
- fix-way-forward, LOW, a crashed producer's lock: fixed with own change. The premise that a stale lock blocks is false: `internal/lock/lock.go` wraps `gofrs/flock`, which the kernel drops at holder exit. The shed-status clause, which a crash can leave `running`, is the part that could block, so it is dropped and the verb gates on the lock alone.
- fix-proposal, MEDIUM, waking a halted run's leftover strand beside the arbiter: fixed with own change, see F5 above.
- fix-proposal, LOW, retries and re-entries had no distinct attempt number: fixed as proposed, D12 gives every attempt the next number.
- fix-proposal, LOW, the seam's refusal blocked advisory readers: fixed as proposed, the decision-notice guard and merge-in's warning fall back to the setting and log the reason.

## Disputed

None.

## Deferred

None.

## Rulings

None.
