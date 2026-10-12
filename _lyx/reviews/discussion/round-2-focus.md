---
round: 2
focus:
  - "Kept-key byte fidelity: `ApplyPatch` in `internal/boardengine/task.go` marshals the existing `Task`, unmarshals into `map[string]any` and re-marshals; settle whether the decision record requires a kept value (for example an integer above 2^53) to survive an upsert, a batch upsert and a merge unchanged, and whether an acceptance criterion pins it."
  - "Kept-key output surface: `outputSuccessWithTask` in `internal/boardcli/cli.go` prints the whole `Task` for promote, upsert and merge, besides `get` and `list-full`; settle whether D4, the Scope line for `internal/boardcli/doc.go` and the matching acceptance criteria name the same set of verbs."
  - "Sweep cost bound: `Root.newRun` in `internal/lyxtemp/run.go` calls `Sweep(true)` before creating its run directory; settle whether D7's Bound states the worst-case latency one `newRun` can add when a dead directory's removal fails for a reason other than the lock-file recreation race, and whether an acceptance criterion pins it."
  - "Retry clock: `internal/lyxtemp` tests build every `Root` with a frozen `fixedClock` for `now`; settle whether D7 says the retry budget's clock and sleep are separate from, or advance together with, that `now`."
---

Targets the round after round 1.
