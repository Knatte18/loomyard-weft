---
round: 1
focus:
  - "Board entry's `## Idea` names two conditions (the record's pair is gone, or the record came from a batten cancel), while the change names the cancel only through the record's carried command: settle whether the diff does what the entry asks for a record written by `lym fabric remove` whose pair is gone, whether the way forward it still gets can finish the teardown, and whether an `Added after Discussion` entry records any narrowing."
  - "Does `lym batten cancel <slug> --remove`, run from the state a killed teardown child leaves (pair gone, possibly the batten run, claim or task branch already removed), finish the remote steps and clear the pending record? Trace the path in `internal/battencli/cancel.go`, including its refusals such as `nothingLeft`, rather than relying on the doc comments; also check what the tests prove about this."
  - "Is `cancelCommand(slug, true)` the right command in every path that reaches `removePairAndBranch`, which already receives a `rerun` argument? Compare what each caller passes there and which flags or command the user ran, and check that the held-lock refusal and the record name a command that works."
  - "Other production paths that reach `pairteardown.Request` or `Topology.Remove`: does any of them have its own retry command, or a removal with no batten run behind it, so that the empty `Rerun` fallback leaves a way forward that answers `pair not found`?"
  - "Dropping the `when origin still holds a branch of the pair` condition and replacing the re-run in `rerunWayForward` and `recordsArchiveWayForward`: check every caller of both (failed-step notices as well as the pending list) for whether the caller's command retries that particular step, including when a later run or record for the same slug is involved."
  - "`## Merge danger` and the change description: check the `two-way` call and blast radius against what the diff changes in the job and pending-record format and in other packages (`pairteardown`, `battencli`), including how another binary reads or writes a `rerun` on a `remove` job, and check the claims under `## Tests` against the tests."
---

Seed pass: no round has been judged.
The entries point at where the diff meets the board entry's wording, and at whether the named re-run works from the state it is printed in.
They do not narrow the rubric's base review of the whole range.
