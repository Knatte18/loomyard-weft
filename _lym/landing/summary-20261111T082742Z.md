# Webster: no card amend, no ownerless work, a truthful verify halt

This change fixes three faults in Webster's run and verify path, reported in #97, #101 and #106.

Closes #97
Closes #101
Closes #106

## What changes

- **A card fix lands as a new commit (#97).**
  An implementer fork that fixed a `lym gate card-verify` failure folded the fix into the card's commit by amending it.
  A status push had already published that commit, so the next push found the task branch diverged and the run stopped.
  The implementer body now says a fix made after a card's commit lands as a new commit on top.
  Its subject is the card's own subject followed by ` (fix)`.
  The body also forbids amending, squashing, rebasing or resetting a commit already made.
  Card tracking reads only the batch's head SHA, so a second commit per card changes no bookkeeping.
- **A Webster session whose run ended does no work (#101).**
  A Master session that outlived its run woke on a decision notice and drove the bracket verbs with no `lym webster run` owning them.
  - A bracket verb refuses when a Master strand calls it while `run.lock` is free.
    The bracket verbs are begin, await, record and recover-batch, `fix-verify` and `fix-plan`.
    The refusal carries `run_ended: true`, and its way forward is to end the turn.
    `webstercli.refuseOrphanedMaster` wraps those verbs.
    Any other caller, such as manual driving from a shell, still gets only the warning.
  - When the run is not running, `lym loom decision add` holds its notice back from the run's Webster Master session.
    The envelope names each held-back strand and the reason under `notice_held_back`.
    A re-entered run renders every recorded decision into its fresh Master prompt, so nothing is lost.
  - The Master prompt gains a rung for the `run_ended` refusal.
    It also says that any message arriving after `outcome: stuck` is written starts no work.
- **The verify-gate halt counts and names its failure (#106).**
  The old reason read `verify gate failed after 4 attempt(s) of 3: none named`.
  It counted gate evaluations against a re-prompt budget, and an evaluation is numbered one past the budget once the budget is spent.
  - The stuck reason now counts the failing entry's own re-prompts against its budget.
    When the run ended with re-prompts left, it says why, in the form `1b6e3fca` gave darn's gate.
  - It names the verify command and what failed: the hang, timeout, failing identities, dirty paths, or the failing run-level entry and its findings.
  - A failure whose log names no failing test now says so and quotes the output tail, cut to 400 bytes.
    That covers a build error or the gate tool's own error.
  - It ends in a way forward: read the log and commit a fix on the task branch, then re-enter.
  - The verify-gate report gains `command` and `exit_code`, and it keeps the log tail for a failure that named no test.

## Design decisions

- #97 is fixed in the implementer prompt only.
  The record's fallback, where batten settles an amended copy with a leased force-push, is not built.
  The amend came from the fork's own fix path, since no Go code amends a card commit, so stopping it at the source leaves no divergence to settle.
- The bracket-verb refusal keys on the caller's strand role.
  So a run that is not live refuses its own Master session, while the sandbox suite's sanctioned manual driving keeps working with only the warning.
- The decision notice is held back only from the Webster Master role.
  Other live strands of a halted run, such as a darn writer waiting on its parent, still get it, since their re-entry may resume the same session.
- The findings Merriam reads keep their `attempt N of M` line.
  Only the halt reason changes, since that is where the count misled the operator.

## To check by hand

None of the following was run against a live run:

- After a tapestry Webster run stops stuck with its Master pane still open, `lym loom decision add` reports that pane under `notice_held_back`, and the pane stays idle.
- A `lym webster record-batch` typed into such a pane refuses with `run_ended`.
- A Webster verify gate that spends its re-prompts on a build failure halts naming the command, the exit code and the output tail.
