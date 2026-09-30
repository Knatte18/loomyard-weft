Goal: the hub orchestrator drives every task with `lyx batten run <slug>` from the prime instead of hand-running `fabric add`, `loom start --no-attach` and `fabric remove`.
Three gaps block that today.

1. **An awaiting child is a planned hand-off, not a halt.**
   After `operator-surface`, a child waiting at Publish for PR approval is `awaiting`; its card 13 makes batten's `Run-Shed` (`internal/battenshed#innerRunProducer.Call`) hard-error on it, failing the batten run and adding a third manual resume.
   Wanted: `Run-Shed` keeps watching an `awaiting` child (self-route, like `running`), so the flow is batten in the background -> orchestrator reviews and runs `lyx loom approve` -> child re-stepped -> batten sees `done` -> `Worktree-Teardown`.
   `blocked`, `failed` and `paused` stay halts.
2. **Teardown destroys the run's records.**
   Friction notes (`loomengine.LoomFrictionDir`, under `.lyx/loom/friction/`) and the driver report (`internal/loomcli/driverreport.go`, under `.lyx/shed/self/`) are machine-local scratch and vanish with the pair.
   Wanted: both live under `_lyx/` and are committed by Go at a loom boundary (same route as `_lyx/reviews/`, `loomcli#statusCommitPathspec`), so they survive teardown on the pushed weft branch.
3. **The watch budget counts approval time.**
   `Run-Shed` has `max_bounces: 1440` at `poll_interval_s: 30` (12 h, `contracts/recipes/batten-recipe.yaml`); a PR left for approval overnight exhausts it.
   Wanted: time spent `awaiting` does not consume the budget (or has its own, much larger one).

Out of scope: deploy (stays with the hub prime after landing); opening VS Code (`batten-open-ide`).