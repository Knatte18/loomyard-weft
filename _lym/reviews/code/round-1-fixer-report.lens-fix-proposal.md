```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F2, F3, F4, F5, F6, F7, F8).

**Findings**

1. **MEDIUM** — F8, `internal/fabriccli/anchor.go:318` and `internal/pairteardown/claim.go:102`.
   - **Summary:** the added "`lym tender pause <slug>`" advice may not do what these two messages ask of the operator.
   - **Evidence for anchor.go:** the check lists a run as live while a batten process is registered (`anchordeps.go:198-211`, `hasProcessRole(records, runproc.RoleBatten)`). `lym batten pause` makes the lifecycle exit at its next boundary, which clears that registration. `lym tender pause` is documented as "hold agent and network work and interrupt running turns" (`tendercli/cli.go:93`), and nothing there says the batten process exits. If it stays registered, the operator follows the new advice and gets the same refusal.
   - **Evidence for claim.go:** the message fires on `claimHeld`, which means batten's state reads running. It tells the operator to "turn a running run into an abandon". A tender pause entry holds dobby but does not change batten's state, so the claim would stay held.
   - **Not confirmed:** I did not read the tender pause implementation. Please verify before keeping the advice. If tender pause leaves the batten process or state running, drop the added clause from both messages and keep it only in `orch-role.md` and the `batten pause` help, where the purpose is "hold against dobby".

2. **LOW** — F4, `internal/battencli/dobbyreads.go`, `unreadableReachedParent`.
   - **Summary:** the episode is cleared only by a successful read of a paused run.
   - **Evidence:** the closure returns early when `run.status.State != StatePaused`, before `Recover`. Suppose a run's judgment fails, the run resumes or finishes, and it is later paused again. Its next failure then continues the old episode at Debug, with no Warn.
   - **Also:** the package-level `EpisodeSet` is process-wide. A slug is unique per hub, so the key is sound.

**Challenged and found sound:**
- **F5:** the new subtest fails under the old `State != running` cleanup, because the lift is dropped while the driver lock is held and the third tick then spawns nothing.
- **F7:** the removed `OriginOwed` and `CancelMarker` paused rows duplicate state-independent checks in `planPass`. The new `OwnedElsewhere` and `Held` rows each hit a distinct branch.
- **F6, F3:** both read true against the code.
- **F2:** the `unreadablekid` fixture was dropped because it read `true`, so the Warn-and-leave path in `DobbyRuns` has no test. This is a known gap, not a defect in the proposal.

**Tool calls**
1. `grep HubBattenLive` and tender pause `Short:` text: to find how the anchor check decides a run is live and what tender pause claims to do. My context lacked both.
2. `Read anchordeps.go:190-230`: to see the `HubBattenLive` implementation. My context lacked it.
