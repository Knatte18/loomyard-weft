# Webster: no amend of pushed cards, no lockless work, named gate failure

Fixes three faults in Webster's run and verify path.

Closes #97
Closes #101
Closes #106

## What changes

**A pushed card commit is never rewritten (#97).**
The implementer stencil (`webster-body-implementer.md`) gains a step: a fix made after a card's commit lands as a new commit on top.
Its subject is the card's subject followed by ` (fix)`.
The stencil forbids amend, squash, rebase and reset on a commit already made, since a status push may have published it.
The fix is in the contract only.
Batten's divergence handling is unchanged, so a run that does rewrite a pushed commit still stops on the diverged branch.

**A Webster session whose run ended no longer acts on mail (#101).**
- `webstercli` wraps the bracket verbs (`begin`, `await`, `record`, `recover-batch`, `fix-verify`, `fix-plan`) in `refuseOrphanedMaster`.
  A call from a Master strand refuses while `run.lock` is free, instead of only warning.
  The refusal carries `run_ended: true`.
  A failed lock probe refuses without that flag and tells the caller to re-run the verb.
  Callers that are not Master strands, such as the sandbox suite's manual driving, still get only the warning.
- `lym loom decision add` no longer wakes a live Webster Master strand of a halted run.
  It lists the strand under a new `notice_held_back` field on its result, with the reason.
  The fresh session a re-entry starts carries the decision in its prompt.
- The Master stencil gains rungs for `run_ended`, for the probe-failure refusal, and for a notice that arrives after `outcome: stuck`.
  In each case the session ends its turn without work.

**The verify-gate halt counts and names its failure (#106).**
- The stuck reason counts re-prompts against the entry's re-prompt budget, as `after N of M re-prompt(s)`, so the two numbers agree.
  When the run ended with re-prompts left, it says why.
- The reason names the verify command, quotes the tail of its output (bounded to 400 bytes), and ends in a way forward.
- When the command failed without naming a test, the reason gives the exit code and says the gate itself may have erred.
  It says so plainly when the output was empty.
- `VerifyGateReport` gains `Command` and `ExitCode`.
  A failure with no identity is reported by its log tail, with no identity list and no card hint.
- The findings text the fixer reads now prints `evaluation N; the session may be re-prompted M time(s)`.

## Design decisions

- `Cap` now means the entry's re-prompt budget, not an attempt cap, so a failed evaluation's number can run one past it.
  The wording follows the form of commit `1b6e3fca`, which fixed the same zero-attempt wording for darn's gate.
- Refusing is limited to Master strands, because the bracket verbs have sanctioned lockless callers.
- Held-back notices are reported on the success envelope rather than as an error, so `decision add` still records the decision.

## To check by hand

- `renderVerifyGateFindings` in `internal/websterengine/verifygatereport.go`: the header line that printed "attempt 4 of 3" was reworded after the last review.
  No fresh reviewer has seen the result.
- The Master stencil's new rungs in `contracts/stencils/webster/webster-template-master.md`, read against a real halted run.
  Only unit tests ran them.
- Whether Batten should also settle an amended copy of a pushed card commit itself with a leased force-push.
  #97 named this as the fallback, and this change does not add it.
- No regression check ran for this task.
