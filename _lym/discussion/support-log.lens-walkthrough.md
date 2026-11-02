```yaml
lens: walkthrough
outcome: reported
```

Lens walkthrough: each decision run end to end on one concrete case, checking that each step's input matches what the step before it produces.

**Findings**

- **BLOCKING, D4 (card fixer re-run):** the fixer's own gate call can join the outer detached run and deadlock until the fixer timeout.
  - Case: the fork calls `lym gate test --gate card --card 3` and the result is red. The detached runner spawns the fixer. The fixer then runs that same command from the worktree root, as D5 asks.
  - `detachKeyDir` hashes the verb, the call directory and the words (`internal/gatecli/detachfront.go`, around line 304). The fixer's call therefore gets the same key as the fork's call.
  - `joinable` lets a call join when the lease holds, the runner is live and the tree identity matches. The tree identity is HEAD's tree plus the list of dirty paths.
  - The card fixer does not commit, and most of the time it edits files the card already dirtied. So the identity usually still matches, and the fixer's call joins the outer run.
  - The outer run is blocked waiting for the fixer, so the fixer waits on itself until the 30-minute timeout. The fix is then lost and the fork gets red.
  - D4's bound only says a `gate-fixer` strand never spawns a fixer. It says nothing about joining.
- **MEDIUM, D4 step 3 ("the caller's repeated calls join the run"):** a fixer that touches a file the card had not dirtied changes the dirty-path list. `joinable` then refuses the fork's next keep-warm call, which starts a fresh gate run on the half-fixed tree.
  - That fresh run is red. If it ends before the first fixer, it spawns a second fixer into the same worktree while the first is still working.
  - "At most one fixer per red call" holds, but the fork never sees the result of the first fixer, so the purpose is missed.
- **MEDIUM, D1/D2 (restart attach):**
  - Case: the driver dies while the wrapped Darn-Write verify closure is blocked on a live fixer.
  - After `AttachGated`, `gateAtBoundary` is false. The loop only re-evaluates on a new Done arrival, a pending entry or an unsent re-prompt (comments in `wait.go` around line 360 and `run.go` around line 440).
  - The writer is idle, so nothing re-evaluates until the row deadline finalize, which can be 120–480 minutes later. Only then does the wrapped closure attach to the fixer record, which by then has timed out as dead.
  - The acceptance line "attaches to that strand and spawns none" technically holds, but the restart stalls the row for hours. The record states no bound for this.
- **MEDIUM, D1 vs D6 (record ownership):** two decisions claim the fixer's record.
  - D1 says `gatefixer` saves the fixer record before the wait.
  - D6 keeps webster's `VerifyFixers` state, which `fixverify.go` saves under the state-mutation lease and fabric-commits as "fix-verify spawn".
  - Walking fix-verify on the shared core produces either two records of one strand, with two attach sources that can disagree after a reset that archives webster state, or D1's save does not apply to webster. The record does not say which.
- **LOW, D7 (Discussion-Write):** the lens-review procedure has a step of its own that runs `validate-discussion` again until it passes, and the fork boilerplate says "passed the self-check before you were spawned".
  - D7 names the Step 6 loop but not those two places.
  - With the fixer on, the writer still loops at full context in the lens-review step unless that step changes too.

**Decision by decision**

| Decision | Case | Purpose |
|---|---|---|
| D1 | webster-style spawn, then done on the reply file | met; restart path partly missed (see above) |
| D2 | Darn-Write: clean passes, verify fails, fixer commits `fix:`, verify re-run passes, Done, producer commit | met. The clean gate runs before verify in the same evaluation, and the blocking closure avoids the pending re-run problem |
| D2 | the same row, fixer leaves a design-class failure | met: findings plus reply go out through `DarnGateReprompt`, and the attempt is counted |
| D3 | `darn.gate_fixer` resolved through `Env.Setting` | met: `Setting` is wired once per loom wiring (`loomcli/wiring.go` line 850) |
| D3 | `card` template `fixer` under a hub `gates` map | met, given per-entry key-by-key merge |
| D4 | fork's red card call | not met: deadlock or re-run split (above) |
| D5 | Plan-Write fixer edits `_lym/plan`, uncommitted, re-run passes, row commits | met, assuming the Plan-Write row commits `_lym/plan` after Done |
| D6 | fix-verify through the shared core | ambiguous record ownership (above) |
| D7 | Plan-Write with the fixer on: writer ends, gate fails, fixer, pass | met |
| D7 | Discussion-Write with the fixer on | partly met: the lens-review "until it passes" loop is not named |
| D8 | row record under the `.lym` scratch dir, role `gate-fixer` | met |

**Tool calls**

- Grepped `detachfront.go` for the detach key and the join rule, because my context had only `detachrun.go`'s head and not how a call joins a run.
- Read `detachfront.go` lines 296–330 to confirm `detachKeyDir` and `joinable`, which my context lacked.
- Grepped the `detachKeyDir` call site and `runDir` to learn whether the key includes a per-strand directory, which my context lacked.
- Grepped `Setting` in the loomcli wiring to confirm darn rows get `Env.Setting`, which my context did not show.
