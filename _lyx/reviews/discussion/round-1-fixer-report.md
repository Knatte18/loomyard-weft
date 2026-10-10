# Fixer report — discussion, round 1

Every finding was checked against the code and fixed; none is disputed or deferred.

## Fixed

- **F1** (BLOCKING): checked against `classifyDeadlineExpiry` and the run-deadline exit in `internal/shuttleengine/wait.go`, which yield `OutcomeDone` whenever every output file exists.
  D1 now states that the deadline ends a held run `OutcomeDone`, with the gate run once at finalize and the shell in `EndedShells` with its result unread.
  It also gains a Bound paragraph explaining why, citing `PATTERN-completion-signal`.
  The first Open risk and the second D1 acceptance criterion now name that outcome instead of `OutcomeTimeout`.
- **F2** (BLOCKING): D6 now enumerates every GOOS and GOARCH pair from one closed list in planglyph, taken once from `go tool dist list` and pinned.
  `unix` and the GOOS implications come from the GOOS through the matcher and are never free tags, and the cap applies to free tags, which D6 now defines.
  The duplicate closed-list sentence, its Notes entry and the auto-mode assumption are reworded to match.
  A new acceptance criterion says `!windows` beside `unix` stays `glyph-ambiguous`.
  The support log's Rejected alternatives records the old mentioned-plus-one enumeration.
- **F3** (BLOCKING): checked against `contractFileStatus` in `internal/websterengine/contractevidence.go`, which reads both the Master and fork slices.
  D3 now counts recovery writes alongside fork writes in the contract-file evidence, and its Bound sentence says so.
  The auto-mode assumption, interview answer 4 and ledger row 5 now agree with it.
  The acceptance criterion now says a recovery's `outcome.yaml` write after Master's last write leaves the finding standing.
- **F4** (MEDIUM): checked against `ownTrackedPaths` and `ResetDeps.OwnPaths` in `internal/websterengine/reset.go`.
  D3 now says that a reset discards, without the dirt refusal, tracked paths a recovery of this run wrote successfully, and that the exemption stays by path and on successful writes only.
  The Bound covers this case, and a new acceptance criterion pins both the reset that now passes and the one that still refuses.
- **F5** (LOW): checked against `gateRepromptText(findingsPath)` in `internal/shuttleengine/gate.go`.
  D2's entry now supplies a renderer, a function of the findings path that returns one line, and that line is validated at send time.
- **F6** (LOW): Scope item 5's probe results and #511 history are cut, since interview question 7 already records them.
  The requirement they carried, no resolver change, stays in D5, which now gives its reason without pointing at the cut text.
- **F7** (LOW): the partitioned-member acceptance criterion now also covers the Rename-new done-check and a plan-gate member read into its file.

## Disputed

None.

## Deferred

None.
