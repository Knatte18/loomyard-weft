# Support log — gate-runs-past-failing-tier

## Interview

A darn run holds no interview: the darn writer read the board entry and the code and made every call itself.

## Rejected alternatives

- **Keep stopping at the first failure and list the unreached tiers and guard step.**
  Rejected: it states the gap but leaves it unchecked, so the agent still covers it by hand, which is the friction the issue reports.
- **Run later tiers only when the failure is present at the base.**
  Rejected: telling a pre-existing failure apart needs a base run of the failing tier, which doubles the cost of every failing gate, while running every later tier needs no such judgment.
- **Exit with the last or the highest failing code.**
  Rejected: the documented contract is the failing tier's own code, and the first failure keeps it for the one-failure case.
- **Continue past a refusal too.**
  Rejected: a refusal's stdout is one JSON envelope, and a second run's output would break it.

## Review rounds

_No rounds yet._

## Question ledger

- **Run past the failure, or report what was not reached?** Self-pick: run past it, since then nothing is left unreached.
- **Which code does a call with several failures exit with?** Self-pick: the first failing run's, in plan order.
- **Does a timed-out tier stop the call?** Self-pick: no, it is a failed run like any other; only a refusal, a busy hub and a signal stop the call.
- **Does the verb print an overall verdict?** Self-pick: yes, one closing stderr line naming each failed run and its code, printed only when a run failed.
