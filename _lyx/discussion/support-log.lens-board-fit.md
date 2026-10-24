```yaml
lens: board-fit
outcome: reported
```

Task: board-fit lens review of the umbrella-groundwork decision record against the other board entries.

**Findings**

1. MEDIUM, D10 against `loom-beam-darns`. D10's promote cannot serve the beam-darns flow, and the record does not name that entry. `loom-beam-darns` step 4 has "the next producer" turn Task A's worktree into a beam *while Task A's run continues*: its darns merge back, then a whole-task perch and the rest of the tail run. D10 makes promote "end a running run for good; nothing of the loom run survives but its commits", turns the entry into a project and clears `recipe` and `depends_on`. So promote either needs a mode that keeps the run, or the record should say that `loom-beam-darns` needs a different primitive and that D10 is not it. Evidence: the `loom-beam-darns` body ("4. The next producer turns Task A's worktree into a beam … 6. … a Task A-wide perch … 7. … PR, landing") against D10's Bound line.

2. MEDIUM, D13 against `umbrella-daemon`. D13 adds new routing for the Land row's ready notice: each newly ready task is named to its own parent's orch. `umbrella-daemon` (operator decision 2026-10-19) removes the "newly ready" part of the notice and the orch's start-on-notice step once dobby lands, and says every batten notice is to be revised. That makes this work throwaway at best. Either cut the per-task ready-notice routing from D13 or name `umbrella-daemon` as its successor. Evidence: the `umbrella-daemon` section "Readiness without batten notices".

3. MEDIUM, D11 and the D1 profile refusal against `run-profiles-ladder`. `run-profiles-ladder` is running, and it changes Seed-Child's writing of `profile` and `run_settings`. D11 changes Seed-Child to seed recipe `beam` for project entries, and D1 refuses profile and run_settings on projects. Both tasks edit the same Seed-Child code, and the record names no ordering or dependency. Evidence: the board list shows `run-profiles-ladder` "running · Worktree-Create", and its body says "Batten's Seed-Child writes only the block of the recipe the task runs".

4. MEDIUM, D13 against `creel-agents`. D13 rewrites notice delivery (`NotifyPrime` → nearest orch). `creel-agents` names `orchcli.NotifyPrime` as the destination of shuttle's hold notice, and plans to move parent asks to creel. The record does not say whether the hold notice (`shuttleengine/hold.go`) and other `NotifyPrime` callers also walk the chain. If they don't, a beam run's hold notice still reaches the prime while its batten notices reach the beam orch. Name the other callers' fate, or mark `creel-agents` as the owner. Evidence: the `creel-agents` body says "The hold notice goes to the prime orch (`orchcli.NotifyPrime`)".

5. LOW, D8 against `batten-window-default`. D8 opens a `--window` run "in the acting worktree's reed session". `batten-window-default` specifies "its own tmux window of the prime's reed session". The two entries disagree on the session, so whichever lands second must reconcile. Name the dependency, or state that D8 supersedes that line. Evidence: `batten-window-default` body.

6. LOW, D9 against `umbrella-anchor`. `umbrella-anchor` (operator, 2026-10-24) plans `lyx beam create --anchor <dir>`, with the default anchor inherited from the parent. Leaving it out of scope is correct, but it says the seams are cheapest touched "together with or just after `umbrella-groundwork`" (`add.go`, `origin.go`, `warplayout.go`, teardown). The record could note that the origin record now carries `beam` (D12) and that D5's `Parent` is where an anchor would be inherited. This is informational and nothing needs to change.

7. LOW, D11 against `landing-escalate-arbiter`. The beam recipe's Finalize fixer escalations go straight to the orch. `landing-escalate-arbiter` plans arbiter-first for Finalize fixer escalations, and the beam recipe will inherit whatever it lands. No conflict, but the Open risks entry "Finalize then halts for the orch" will go stale once that lands.

8. LOW, Scope's out-list against `umbrella-daemon`. The out-list rightly gives `lyx beam start` and dobby to `umbrella-daemon`. That entry's "Draft tasks" decision adds a board `draft` field in the same board code as D1–D3. The two don't conflict, but whichever lands second touches the same `Task` fields, README layers and upsert rules. Naming it would help sequencing.

No duplicate of other planned work found. The board, fabric, batten and orch parts are not claimed by any other entry; `parallel-batch-parts` and `weaver` only consume D5 later, as the Scope says.

Tool calls:
- One `lyx board get` of `run-profiles-ladder`, `batten-window-default`, `creel-agents` and `weaver`, filtered to beam, orch, notice and batten lines. My context held only these entries' titles and briefs, not their bodies.
