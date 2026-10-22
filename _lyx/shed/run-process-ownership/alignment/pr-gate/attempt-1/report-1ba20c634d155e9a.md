---
verdict: APPROVED
---

## Alignment at PR-Gate — run-process-ownership

Pull request 837 at `366897121` against `main` (`31579d6ca`, also the merge base) matches the board entry's goal and the decision record's D1 to D11.

### Goal and decisions against the diff

- D1 and D3: `internal/runproc` holds the owner record (`record.go`), `End` (`end.go`) and its leaf test; `PATTERN-leaf-packages` and the new `PATTERN-run-process-owner` record it.
- D2: registration at `fabricengine/spawn.go` (`SpawnDetachedPush`), `boardengine/spawn.go` (`SpawnSync`), `loomcli/drivewiring.go` and `looparm.go` (drive and the detached loop), and `battencli/arm.go` (the seeded batten).
- D4 and D5: `pairteardown` (`skip.go`, `teardown.go`, `claim.go`) ends owned processes after the refusal probe and before the loop and session end, with the skip readings and the run-lock wait first; `fabriccli/remove.go` runs the run-scope reconcile and removes `ended` and `owned_elsewhere` pairs in local mode; `reedcli/up.go` ends the pair's processes on local readings only; `pauserecord/hold.go` adds the waiting marker and the remove process's gate exemption.
- D6: the hidden no-wait flags and the parent-lock spawn flag in `fabricengine` and `boardcli`, and `tenderengine/resume.go`'s `spawnOwed` in the shared lift sequence.
- D7: `tendercli.PauseOwnedElsewhere` ends after the owner pause and spares a Finalize past its parent-side merge through `battenshed.FinalizePastMerge`; it is wired into batten's owner probe, `lyx orch start`, and the hibernate wake through `EndOwned`.
- D8: `tenderengine/processes.go` and `reap.go`, and the `reap` verb in `tendercli`.
- D9: reed's session end is untouched.
- D10: `hibernateengine/watch.go` and `request.go`.
- D11: `shuttleengine/wait.go`.
- Docs: `runproc/doc.go`, each touched package's `doc.go`, and `docs/overview.md`'s module table and tender line.

Out-of-scope items stay out: no cancel verb (`batten-cancel-verb`), no spawn audit log (`spawn-audit-log`), no change to reed's session end or the hub-level daemons.

### Carried-over open risks

- Discussion-Review `d6-push-child-coverage-reading` and `d7-revival-does-not-restart-driver`: design risks, closed by an accepted circling decision; D6 and D7 record the chosen behaviour and the diff follows it.
- Plan-Review `await-run-lock-or-pass-refusals-unreached` and `reed-down-retry-step-mismatch`: fixed in the plan's round-4 fixer report, and the diff carries `awaitrunlock_integration_test.go` and the retry-step wording.
- Webster-Review `runproc-end-group-members-not-watched`: fixed in `558b1d880`, where `End` watches every live member of a signalled group.
- Webster-Review `pattern-md-unrelated-entries-shortened`: four existing `PATTERN.md` overview lines (`PATTERN-owner-machine`, `PATTERN-glyph-conversion-chokepoint`, `PATTERN-verified-tree`, `PATTERN-pause-holds-prompts`) lose one clause each so the overview stays under its headroom line after the new entry is added.
  Each dropped clause is still in its background file, so no invariant changes; this is bookkeeping the headroom rule forces, not a scope deviation.
  Agents that only get the overview inlined no longer see those four clauses there; the operator may want board entry `plan-gate-pattern-headroom` to weigh that.

### Related board entries

- `fabric-remote-async`, a note, says no lyx command waits on the network, apart from the branch-exists probe.
  D5 adds a synchronous run-scope origin reconcile to `lyx fabric remove`, and the decision record's Constraints explain why: `PATTERN-origin-reconcile` requires it, and it is an origin read like the probe that entry keeps synchronous.
  This is a recorded decision, not a conflict with a design being built, so I did not escalate it.
- `batten-cancel-verb`, `orch-strand-cycle`, `loop-exit-halt-names-its-cause`, `done-check-spaced-create-paths`, `fork-git-guard-read-only-tag` and `retry-archive-tag-push` depend on this task and are not in flight.
- `spawn-audit-log` and `umbrella-daemon` build on the owner record later; this run does not duplicate either.

### Overlap and landing order

`lyx loom peers` lists no peer run and no parent commit (`own_row` 21).
There is no overlap, so no landing order applies.

### Superseded

None: this run's notes were folded into its own entry, and no other board entry is made obsolete.
