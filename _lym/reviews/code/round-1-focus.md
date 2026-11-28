---
round: 1
focus:
  - "`firstPassExited` and the catch-up owe/run path: when the dobby's own catch-up succeeds but the task's first pass keeps exiting `PassExitOriginWait`, how often is the task started again, what bounds it, and does anyone get told? Trace every producer of that exit code, since it now bypasses the start record, notice and backoff."
  - "`oweCatchUp` and `catchUp` where `Deps.CatchUp` is nil or has no `PullBolt`: what does an origin-wait first-pass exit do there, and when is the task retried?"
  - "The `catchUpOwed` guard in `startReady`: does anything else that function does, besides starting tasks, need to keep running while the catch-up is owed, and do all its callers and the existing tests agree with the early return?"
  - "Lifetime of `State.CatchUp` and `Since`: across success, a change of reason, an alternation between reasons, a dobby restart and a record nobody retries any more, can the push summary report a wrong duration, post a premature or missing all-clear, or report a wait that has ended?"
  - "`summarisePushes` with the catch-up wait: check how it combines with the owed period, the threshold, the anchor-change stop and the held entries, and whether `earliest` makes the notice and the all-clear fire as the doc comments state."
  - "Does a catch-up waiting on origin reach a person in every dobby that can wait on it? Check the change description's claim that the prime's dobby reports a beam dobby's episode: what makes the prime's catch-up owed during that outage?"
  - "Exit paths that map to `PassExitOriginWait` for a refused registration or another failure: does the dobby's own catch-up then post the notice the change description says it does, for each of them?"
  - "The amended goal asks for the start to be retried on the dobby's backoff and reported only past a bound, as pushes are: does the diff's behaviour match that for the retry delay and the reporting bound, and do the package doc, specs and `pattern/` entries that describe start refusals or catch-up waits still hold?"
  - "New subtests in `starts_test.go` and `pushsummary_test.go`: do they fail without each behaviour they name, including the `startReady` guard and the same-tick exit, the carried `Since`, and a record written without it?"
  - "Change description's `## Merge danger`: check the door and blast radius against the new persisted `since` field and every reader of the dobby's state file."
---

Seed pass: no round has been judged yet.
The entries are open questions drawn from the committed diff and the change description; none states a verdict.
