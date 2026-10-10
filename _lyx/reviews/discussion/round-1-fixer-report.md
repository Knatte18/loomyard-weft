# Fixer report: discussion round 1

Every finding was validated against the code and fixed in `_lyx/discussion/decision-record.md`, with the matching support-log lines.
`lyx loom validate-discussion` exits 0 over the result.

## Fixed

- F1 (D12): `BatchState` gains `RecoveryStartSHA`, HEAD at each recovery spawn, while `StartSHA` keeps inheriting the bracket's start as `recoverSpawn` does today; `recovery_retry` compares HEAD with `RecoveryStartSHA`, and the acceptance row separates a recovery that committed from a fork that did.
- F2 (D7): the allowlist adds `agentname.RoleOrch`, matched through `agentname.MatchesRole`, since `hubgeom/parent.go` gives a nested pair's orch a slug.
  The verb side is now deny-by-default: every subcommand but the named readers is refused, and a test pins every registered subcommand.
  The Bound states that an unset, empty, unparseable or slug-free name passes, as `configcli`'s `hubWriteRefusal` does.
  A criterion covers the slugged orch name.
- F3 (D8): every git reader's accepted form is listed, with `--output`, `--ext-diff`, `--textconv`, `--upload-pack`/`-u` refused, `branch` limited to named list flags, `cat-file` to `-t`/`-s`/`-e`/`-p`, `config` to `--get`/`--get-regexp`, and `-C <dir>` the only global option.
  The `--help` match must be a whole unquoted word before any `--`.
  The record states that today's refusals stay (redirection, substitution, backgrounding, quoted or unlisted first word), so `env`, `xargs` and `bash -c` stay rejected.
  The criterion adds a reject row for each edge.
- F4 (D12): `recoverSpawn` carries `prior.Recoveries + 1` and saves it with the spawn record; a rebaseline keeps it.
  One premise was wrong: `lyx webster reset --to batch-start` moves the branch and keeps the batch record (only `--to start` archives, `internal/webstercli/reset.go`), so the reset does not discard the count by itself.
  D12 now says the batch-start reset clears the batch's count and saves the state, which the `recovery_exhausted` way forward needs.
- F5 (D12): the only refusal is now the cap: `recovery_exhausted` at two recoveries, for every caller, with a spec row under the existing constraint.
  A dead recovery with no commits of its own is not refused.
  It reports `recovery_retry: false`, so Master ends stuck as before, and an operator can still run the second recovery.
  A dead fork prior respawns as today.
- F6 (D9): a gated run that has sent a gate re-prompt now treats a waiting turn end with files present as a missing-file one, which keeps the leftover-files guard where an earlier arrival exists.
  The Bound states what a stale shell can now hold open and that it never ends a turn early.
  `ShellWaitExpires` is removed and `TurnEndedAfter` counts a turn end only with nothing outstanding; criteria cover the before- and after-re-prompt cases.
- F7 (D1): the rebaseline warning is logged at Warn and joins the entry warnings, so it reaches `RunResult.Warnings` and the summary on done and opens the Stuck reason otherwise.
  The restamped state is saved before the batch loop, and the criterion asserts both.
- F8 (D1, D2): both Decisions now say the gate catches only the done-card edit and that the other three refusals block at the Webster row.
  D2's Rationale is narrowed to match, and criteria cover the in-flight amendment and each listed refusal.
- F9 (Constraints): the friction line now binds D12's webster refusal and says D7's `fabriccli` refusal writes none.
- F10: the Goal drops the tally, and Interview item 1 no longer claims the Goal lacks #476.
- F11: criteria now cover the `add --help` push text, the implementer stencil rule, the fixer-prompt marker, the no-recorded-hashes case, the absent-copy cause, `prune`/`cleanup` without `--apply`, D9's ungated, awaited, fork and deadline cases, the recovery report-present and report-missing cases, and D13's inline span.
- F12 (D7): the import is settled in D7, since `mergeresolve` imports `fabricengine`, never `fabriccli`, so there is no cycle.
  The ledger entry is marked resolved and the `agentname` fallback is gone.
- F13 (D11): the candidates move to the Notes for the plan writer without the timeout values.
  D11 now requires at least twenty repeated runs, a forced-dependency red before the fix and green after, and the documented attempt as the fallback.
- F14 (D5, D8, D9): D5's splitter treats quoted separators as text.
  D8 keeps `IsReadOnlyCommand`'s own quote-aware split and adds `||`.
  D8 and D9 name the pinned rows and doc comment they flip: `TestIsReadOnlyCommand`, the `waiting_test.go` expiry rows and `TestTurnEnded`.

## Disputed

None.
The batch-start part of F4's premise was wrong, as noted under F4, but the finding's core holds (the count is neither carried nor cleared), so it was fixed rather than disputed.

## Deferred

None.
