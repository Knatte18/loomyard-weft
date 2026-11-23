# Red-first gate holds every run in a hub whose test scoper is not go

`red-first-non-go-scoper` · Task · bug, gate, loom, dotnet · starting · Worktree-Create

Depends on: [`verify-budget-notice-no-slug`](design-verify-budget-notice-no-slug.md)

Imported from [issue #192](https://github.com/Knatte18/loomyard/issues/192).

Seen in a C# hub (NORCE Models), darn run hydraulics-gaps: the red-first gate held the writer's handoff because it cannot run the diff's tests on the merge base when the hub's scoper is not go. The run went on only after the parent orch recorded a decision titled 'No test seam ...' by lym loom decision add, waiving the check. Workaround: the parent records a 'No test seam' decision naming why the check cannot run. Expected: in a hub whose scoper cannot run tests on the merge base, the gate skips red-first or runs it through the hub's own test command, instead of halting every run for a manual waiver.