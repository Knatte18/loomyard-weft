# Round 2 fixer report: discussion

I checked every finding against the code and found each premise true.
All nineteen are fixed in `decision-record.md`, and three rejected alternatives were added to `support-log.md`.
`lyx loom validate-discussion` exits 0.

## Fixed

- F1 (D7): a `push` carrying the hidden `--weft-path` or `--warp-path` bypass flag passes the guard, since it is the detached child `Fabric.Commit` spawns (`internal/fabricengine/spawn.go` sets no `Env`).
  The Bound says a fork can type the flags, a bypass push pushes only the named worktrees, and bypass mode refuses every other subcommand.
  Added an acceptance row and an Open risk.
- F2 (D7): the `fabric sync failed` and `decision: commit failed after the append` spec rows now say the orch runs the verb when the refusing session cannot.
  Master's stencil already ends stuck on a fabric-sync failure, and `commit-records` is driver-run (`internal/loomcli/driverprompt.go`).
  The Constraints bullet covers the reworded rows.
- F3 (D7): the guard runs only when `fabric` is the invoked top-level command, so `configcli`'s in-process sync passes.
  Guard tests set the strand name with `t.Setenv` and run without `t.Parallel`, as `configcli`'s guard rows do.
  Added an acceptance row and a Constraints note.
- F4 (D12): a spawn that renders an unrendered amendment, Master's `card_amended` re-run, keeps the prior count, and the cap refuses counted spawns only.
  A new Master rung ends stuck on `recovery_exhausted`.
  Added acceptance rows.
- F5 (D12): `BeginBatch` carries `Recoveries` and `RecoveryStartSHA` over beside `StartSHA`, `AuditWarnings` and `ForkTranscripts` (`internal/websterengine/beginbatch.go`).
  Added a Notes pointer and an acceptance row.
- F6 (D9): the gated exception no longer depends on a re-prompt having been sent.
  It now applies only to a run started fresh, before the gate has evaluated any arrival (a pending `PassOnCap` or `MayHold` answer included), with every outstanding task an unawaited shell.
  An attached or resumed run waits.
- F7 (D9): a fork, an awaited shell or an empty outstanding list holds a gated waiting turn end, so `TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival` stands, and D9 says so.
  The reversed tests are now named `TestPollEventsTick_ShellExpiry`, `TestWait_GatedShellExpiryEvaluatesGate` and `TestRun_ExpiredShells`, and the `TestTurnEnded` outstanding-shell rows are marked new.
- F8 (D8): the classifier is injected.
  `ClassifyViolation` gains a `readOnly` parameter and `RecoverDeps` gains a field, both filled by `webstercli`, so `websterengine` still imports no `fabricengine`.
- F9 (D8): each git form now carries a positive flag allowlist with exact matching, so abbreviations and `--exec` are rejected.
  The `lyx` help form requires command words only, with the help word last.
  Added the rejects to the acceptance list.
- F10 (D1): `Rebaseline` takes the step its refusals name from its deps, so the auto path's refusal says to fix the plan and re-step the row.
  Added an acceptance assertion.
- F11 (D8): a rejected command still refuses `needs_fresh`, and failed evidence refuses with `ErrAuditNotAcceptable` naming `recover-batch`.
  `AcceptBatchFabricReference`'s bound wording is rewritten.
  Added an acceptance assertion.
- F12 (D9): resolved in the F6 and F7 rewrite.
  Ungated runs count files present at once whatever is outstanding, and gated runs are held by forks and awaited shells.
  The criterion is conditioned accordingly.
- F13 (D12): "past" now means a non-merge first-parent commit in `RecoveryStartSHA..HEAD`, and an empty start, a reset-moved HEAD or a range of merge-ins only all give false.
  The count and start are saved before the strand starts, so an unrecorded started strand is still counted.
- F14 (D1): every error the auto-rebaseline returns, transient ones included, maps to Stuck with a re-step way forward.
  A nil plan or nil state stays a returned error, as a programmer error.
- F15 (D5): a backslash-newline continuation is joined before the split, and a `cd` in a pipeline stage or after `||` ends the cross-module scan.
  Added an acceptance assertion.
- F16 (D2): the marker sits on the rubric only.
  The fixer-prompt marker, its acceptance wording and the `burler-step-3-fix.md` Notes pointer are dropped.
- F17 (D3): all three empty-frame causes are named, and the cause is returned beside the frame.
  The acceptance row covers all three.
- F18 (D13): the finding names the cap and the shorten-or-move way forward, and only fenced blocks are scanned.
  The inline-span clause, its acceptance wording and the `codeSpans` Notes pointer are dropped.
- F19 (Acceptance): added the unedited done card, the Stuck reason opening with the rebaseline warning, the D11 no-reproduction branch and the gated files-present cases.
  For D12, added `recovery_retry` false at two recoveries, re-begin and rebaseline keeping the count, and `reset --to start`.

## Disputed

None.

## Deferred

None.
