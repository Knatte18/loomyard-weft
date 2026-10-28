```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checked every claim the fix added or rewrote against the code at HEAD. One finding.

- **LOW**, D2 Bound, added sentence "A program claim is accepted off Linux from any process whose strand name is unset".
  The code accepts a program claim off Linux in a wider case than the record says.
  It is accepted whenever `$LYM_STRAND_NAME` does not parse as a strand of this hub: unset, but also malformed or under another hub's shortname.
  Evidence: `internal/creelengine/identity.go`, `resolveFromEnvironment`, returns `StrandSender` only when `agentname.Parse(claim.StrandName)` succeeds with this hub's shortname, and otherwise returns `ProgramSender(program)` for any claimed program.
  Suggested wording: "whose strand name names no strand of this hub".

All the other claims hold:
- **D8 Bound:**
  - A paused or dormant receiver freezes the TTL (`deliver.go` `locate`: running is live and unpaused).
  - A line is typed only at idle (`typeInject` waits on `w.idle`).
  - creel appends no `MessageTail` (nothing in `creelengine` or `creelcli` mentions it).
  - Only EXTERNAL, programs and orch strands may inject into another strand (`doc.go:201`).
- **D2:**
  - On Linux a program claim is accepted from any process outside every pane, or from an orch pane (`resolveSender`).
  - The CLI has no program flag (`Claim.Program`, `doc.go:49`).
  - The 0700 lymtemp root is the stated boundary.
- **D10:**
  - `sweepSubjects` takes only unread mail and queued injects.
  - The two-miss rule is real (`goneAfterSweeps = 2`).
  - `persistAged` writes every `agedPersistInterval` and whenever the running state changes, so also when the clock stops.
  - A hook strand rings once per claimable set and gets at most one bell per set (`wakeHookStrand`, `typeBell`).
  - `carryRecords` carries only unread messages, queued injects and live holds.
- **D12:** `journalRetention` is 7 days, in `journal.go`.
- **Constraints:** `go list -deps` evidence was confirmed in context; `OrchStrandOf` and `OrchLive` use `orchengine`, `reedengine` and `fabricengine`.

Tool calls:
1. grep of `deliver.go`, the `creelengine` inject-authority text and `MessageTail`: to check the D8 Bound claims, which context lacked.
2. `sweep.go` `sweepOnce`, `deliver.go` persist logic, `fold.go` `carryRecords`: to check the two-miss, aged-persist and retention claims.
3. `identity.go` `resolveSender` and `checkProgramClaim`, plus `goneAfterSweeps`: to check the D2 program-claim claims and the miss count.
4. `deliver.go` `wakeHookStrand` and `typeBell`: to check the hook-strand ring and bell bound in D10.
