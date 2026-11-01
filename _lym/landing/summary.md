# gate: a long gate run answers within the prompt-cache lifetime

An agent blocked on `lym gate test` or `lym gate card-verify` now gets an answer within its prompt-cache lifetime, even when the gate run outlasts it.
Before, only the slot wait was capped; once the slots were held, the call blocked to the end of the tiers and the agent came back to a cold cache.
Now the call answers exit 75 with an ok envelope and `done: false`, the running phase, and the message to run the same command again.
The tiers go on in a detached runner, and the same command again joins that run and prints its stdout, stderr and exit code once it ended.
The keep-warm cap also stops being the constant `shuttleengine.DefaultAwaitCap` and becomes per-role config.

## What changes

- **Detached, joinable runner** (`internal/gatecli`: `detachfront.go`, `detachrun.go`).
  A capped call forms a join key from the verb, the run directory and the argument words (card-verify keys on the card's own id, so `4` and `4-<slug>` join one run).
  It joins the current run or starts a hidden `run-detached` runner under a per-key directory in the worktree's gate tree.
  A join needs a live lease, an undelivered result, an unchanged HEAD tree and dirty path set, and a live or finished runner; otherwise the front supersedes the run and starts a fresh one.
  A recorded exit 75 of the test's own is replayed as exit 1 with one added stderr line, so it never reads as "run again".
- **Which calls detach.**
  Only a call with no `--wait`, no inherited slot, a strand in its environment and a non-zero resolved cap.
  `--wait`, nested calls, `gatecli.RunGate` and the operator at a terminal run in-process as before.
  A call with no strand now waits for slots without a cap.
- **Lease.**
  Each front writes a lease (its cap deadline plus 120 s) that the runner reads at most once a second.
  When it passes, the runner kills its tiers, frees its slots and records the abandonment, so an interrupted or dead agent holds slots for at most one cap plus the grace.
  The runner is registered in `internal/runproc` under the new `runproc.RoleGate`, so pair teardown ends it.
- **Cap config.**
  `claude.yaml` gains `keep_warm_cap` (template `250s`) and `keep_warm_cap_roles`, validated beside the prompt-cache lifetimes: at least 60 s, below the role's lifetime, at most the long cap.
  `shuttleengines.KeepWarm` resolves a strand's cap through `shuttleengine.Engine`; `lym gate test`, `lym gate card-verify` and the default of `lym shuttle await --cap` read it.
  `DefaultAwaitCap` stays as the fallback when the config cannot be loaded.
- **Break-even long wait.**
  New `shuttle.yaml` key `keep_warm_max_reruns` (template `10`).
  For a `--gate` key whose median wall time over its latest five full calls exceeds cap times that value, the front waits with the long cap (55m on the claude engine) in one call.
  Run records gain a call id and an `Abandoned` mark; `lym gate report` counts abandoned records apart.
- **Card-verify** takes the same form.
  `cardverify.Runner` takes the caller's context, ties each line to the runner (parent-death SIGKILL on Linux, a kill-on-close job on Windows) and records the line's process group, which a front that supersedes a dead runner kills.
- **Agent text.**
  Gate calls rendered into agent prompts (implementer card and batch gates, the verify fixers' `fix_gate`) drop `--wait`.
  The gate rule in `internal/shellguard/rows.go` and the implementer, darn and drift stencils read 75 with `done: false` as "run the same command again".
  `lym shuttle await` help and the `DefaultAwaitCap` doc name a 3600000 ms Bash timeout.
- **Strand removed from verify commands.**
  Every `verifyrun` caller (`cardverify.Runner`, `battencli`'s `runDeploy` and `runRowCommand`) removes the strand from the command's environment, so an operator-written command holding a gate call runs in-process.

## Design decisions to know

- Exit 75 with an ok envelope keeps one meaning, "run the same command again", for the slot wait and the run wait.
- The template cap stays 250 s for every role: a fork's Bash shell carries its session's strand while its own cache runs at the 5-minute subagent lifetime, so a role-wide cap above five minutes would leave forks cold.
- The join compares the tree and the dirty path set, not dirty file content; an edit to an already-dirty file between two calls with the same words replays the earlier run.
  The gate rule bars such a turn, and the green record still needs an unchanged clean tree.
- The llm driver's `lym shed step --until-stop` wait and the Webster Master's waits are unchanged, as are Go verify sites.
- On Linux a parent-death SIGKILL reaches a tier's own process but not its descendants, so a runner killed without an exit file can leave a tier's descendants running without a slot.
- `PATTERN.md` gains `PATTERN-keep-warm-cap`.

## To check by hand

- `endStrayLine` kills a dead card-verify runner's line group with `proc.KillTree` on the group recorded in `line.json`.
  That group is owned by its line record, not a `runproc` owner, so it falls outside `PATTERN-run-process-owner`, as the loop step trees in `internal/pairteardown/loop.go` do; check the departure is acceptable.
- Card 10's tests reach unexported helpers; check they exercise the behaviour through its public seam.
- The card 11 change to the `--help` subtest in `internal/gatecli`; check the help output still reads correctly.
- The lease and the current-run pointer sit beside, not inside, the run's state files; check a late write of a superseded runner cannot reach a newer caller.
- Windows paths (`proc.CreationTime` join check, `proc.HoldNamedJob` line tree) are covered only by code and unit tests here; run a capped gate call on Windows.
- A real agent session: run `lym gate test` past the cap and confirm the exit 75, the rerun that joins, and the replayed result.
- The plan-level verify failed once on `claudeengine.TestBuildSettings_MatchesGolden`; the fix updated `testdata/settings-golden.json` to the new gate rule text, and the follow-up gate run on it ended before its result was read.
