# Fixer report — plan review round 1

All seven findings are fixed.
`lyx loom validate-plan` exits 0 after the edits, reporting informational `create-new-unit` findings only.

## Changed

- **F1 (BLOCKING)**, the return to `idle` not retiring what the cycle saw:
  - `08-orch-cycle-machine.md` gains a return-to-idle rule.
    Every return to `idle` (resume completion, resume timeout, handoff abort) persists `LastInjectionOffset` as the cursor read through and forgets the in-memory newest event.
  - A resume timeout also marks the reading unknown.
    The handoff-abort paragraph now points at the rule that enforces D11's "next qualifying idle turn end".
  - Four test cases are added: ticks after a resume timeout inject nothing, a handoff abort does not retry on its already-read turn end, every return persists the offset, and a restart in `idle` after a resume timeout does not re-read the pre-clear handoff turn end.
  - `06-orch-state.md` restates `LastInjectionOffset` to match.
    The overview's `events-cursor` decision now states the return-to-idle rule.
- **F2 (MEDIUM)**, the smoke threshold: `14-orch-smoke.md` step 1 now seeds a threshold no smoke session reaches, so the manual `cycle` is the one trigger.
  It states why a low threshold cannot settle at one cycle, and that card 8's unit cases cover the threshold-triggered path.
- **F3 (LOW)**, the restart tests versus the clearing rule: `08-orch-cycle-machine.md` scopes the landed/not-landed restart pair to `handoff-requested` and `resuming`.
  It adds a restart-in-`clearing` case (exactly one `ClearSession`, then the resume prompt once the probe passes) and says the `clearing` re-clear is unconditional per D10.
- **F4 (LOW)**, the liveness probe starving a watcher: in `09-orch-watch-loop.md`, `Run` retries the watch lock a small, named number of times before returning `ErrWatcherRunning`, with the reason stated.
  The test list gains a lock-released-between-retries case.
- **F5 (LOW)**, the untested Agent-tool override: in `10-orch-cli-skeleton.md`, the override moves into a pure `orchShuttleConfig`, and `cli_test.go` covers it clearing `ClaudeDenyAgentTool` and leaving every other field alone.
- **F6 (LOW)**, card 4's ImpactSummary: it now names `lyx fabric clone`'s reconcile, which commits `_lyx/orch.yaml` into every new hub, `hubforge` fixtures included, and the registry-derived clone-config test.
- **F7 (NIT)**, card 12's ImpactSummary: it now says only `cmd/lyx/helptree_test.go` changes outside the module.

## Deferred

None.
