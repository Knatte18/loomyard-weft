# Support log — adopt-runs-tests

## Interview

A darn run holds no interview; the writer read the board entry, its operator decisions and the adopt and gate code, and made each open call itself.

## Rejected alternatives

- Every tier as the full suite: it would run the manual LLM tier, which the hub runs by hand only.
- Every adopt until the next commit lands as the session: it needs a record of which tree was tested on which machine, which the gate's green record already gives for a carried tier.
- A red suite refusing the adopt: the owner write and the resumes have already taken effect when the suite ends, so a refusal could not undo them.
- Running the suite in the foreground: the adopt would block for the suite's whole run.
- A creel notice on a red suite: it needs a wrapper process that reads the gate's result and a sender identity for it, beyond this task's size.
- Running the suite in each adopted beam: the board entry asks for one run per session, and the prime holds the landed code.

## Review rounds

_No rounds yet._

## Question ledger

- What "full suite" maps to — self-pick: the `finalize` gate key.
- What marks an adoption session — self-pick: one `lyx adopt` call.
- Whether a red suite refuses the adopt — self-pick: it only reports.
- Whether the suite runs in the background — self-pick: yes, detached, with a log.
