# Support log — driver-context

## Interview

The session ran autonomously with no operator; every turn below is a self-pick after reading the board and the code.

1. Read the board entry and the related notes (`verify-watchdog`, `orch-owns-the-loop`, `run-message-routing`, `lyx-halt`, `haiku-roles`).
   Took from them: the step watchdog belongs to this task; the driver's shell can be killed with its process group after a bounded time; the orchestrator messages the driver, never types into panes; the driver is the one LLM alive for the whole run.
2. Read the shed engine, the shed verbs, the loom driver launch and stencil, the retention sweep, batten's notice judgement, and the refusal spec.
   Found that `lyx loom run` is already a Go loop (the `go` driver) without an LLM to wake, that `pause_requested` is the engine's shared write-to-clear flag, that per-step records and traces are keyed by process trace id, and that the `bolt-and-store` silent park came from an arming refusal reported before the verb ran, with no kind, no trace and no state change.
3. Settled the loop as a mode of `step` running a child per step rather than an in-process walk, because the child keeps every per-step hook, record and trace unchanged and gives the watchdog a process to kill.
4. Settled the detached loop plus waiter shape, because of the shell lifetime the board records.
5. Settled the stop condition in the engine, because the engine already consumes the bare pause for both drivers.
6. Settled the watchdog on idleness over batten's activity reading, because a per-step deadline would kill Webster.
7. Settled the trace copy over a retention pin, because the sweep is a leaf that knows no run state.
8. Settled the failure-stop invariant: a failure stop writes `failed` when the status file still reads `running`.
9. Settled the stencil rewrite, the fork, the guide stencil, the skill drop, the status strand and the teardown kill.

## Rejected alternatives

- **Reusing `lyx shed run` as the driver's loop.** It holds the run lock for the whole walk, runs no per-step hooks (loom's bootstrap, voucher and halt note), keeps no per-step records, and its envelope lacks every key the driver branches on; the driver's contract would change for no gain.
- **An in-process loop over `shed.Step`.** One trace id for the whole loop breaks the per-step records; a hung producer cannot be killed from inside, and cancelling its context reads as a pause; stderr noise would still reach the driver.
- **The loop as a plain background job of the driver's shell.** The board records that such a shell is killed with its process group after a bounded time; a loop that dies at Webster mid-run defeats the purpose.
- **The loop as a hidden reed strand.** More moving parts (strand naming, revival, pair teardown through reed) than a detached process, which already has the Go driver as precedent.
- **A retention pin for the stop step's trace.** The sweep is a leaf package; a pin needs a writer, a remover and an ageing rule, where a copy into the steps directory needs nothing.
- **A per-step deadline instead of an idle window.** Webster and the reviews run for hours legitimately.
- **A seventh envelope shape for the loop.** The stop step's own short envelope plus one `loop` key keeps the driver's branches unchanged.
- **The driver running the pause command only at a boundary it notices.** The engine reads the condition at every boundary whatever the driver does, so the producer in progress always finishes and no loop restart is needed.
- **Dropping the Agent tool from the driver (board idea 5).** The one-shot fork needs it; fork mode keeps every other subagent type denied.
- **Re-invoking an interrupted step from the loop.** Webster's `handback` policy exists because a re-invoke double-spawns; the driver's rule and its cap already decide this.

## Review rounds

_No rounds yet._

## Question ledger

- Loop shape (auto-pick): a mode of `step` with a child per step; see D1.
- Shell lifetime (auto-pick): the loop runs detached and the driver's command waits on it; see D2.
  Assumption: the bounded shell lifetime the board records still holds.
- Stop condition home (auto-pick): the status file, consumed by the engine; see D3.
- Retracting a condition (auto-pick): `--clear`, since the only other route is a hand edit of the status file, which the stencil forbids.
- Watchdog criterion and window (auto-pick): idle over batten's reading, 60 minutes default, configurable; see D4.
  Open: the right default is read off the first runs.
- Sixth kind (auto-pick): `interrupted`, named in PATTERN-shed-verb-set; see D4.
- Failure visibility (auto-pick): a failure stop writes `failed`; new PATTERN entry; see D5.
- Pre-step refusal kind (auto-pick): `bootstrap`, as the kind's own doc already describes; see D6.
- Trace retention (auto-pick): copy at the stop; see D7.
- Transient re-step (auto-pick): in the loop, once; see D8.
- Fork at a stop (auto-pick): one fork per error or interrupted stop, returning a conclusion and one action; see D9.
- Guide stencil (auto-pick): rare branches move to a second stencil the prompt points at; see D9.
- Status strand on llm runs (auto-pick): kept, since the driver pane goes quiet; see D11.
- Idea 7 (auto-pick): out of scope, the board marks it optional.
- Environment: nothing outside the two discussion files was touched; no tool failure blocked the steps.
