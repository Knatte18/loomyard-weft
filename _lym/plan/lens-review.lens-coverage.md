```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every decision, acceptance criterion and branch in the plan's overview and cards 1–17 against the decision record. Thirteen findings: two BLOCKING, eight MEDIUM, three LOW.

**BLOCKING — Card 11 (dobby-pass-act): the dobby would start passing halted and blocked runs.**
- Card 11 replaces `drivable` with "`Verdict.Answer` is act".
- Under card 3's `Act`, a halted run and a batten record blocked before Run-Shed (which reads `Starting`) both answer act, because step 6 is "act otherwise".
- `drivable` gave such runs no pass (`internal/dobbyengine/passes.go:71-73`). The card would give one on every probe tick, so the dobby re-drives runs that halted for the operator.
- `TestPlanPass`'s rewritten rows cover only wait, hand-over and "act gets today's passes". No row covers a blocked or halted batten run.

**BLOCKING — Card 7 (cancel-intent), with card 3's `Aside`: cancel refuses a batten-paused run.**
- Card 7 has the cancel ask `Judge` with `Cancel`, `Pause` and `Hold` set aside.
- Card 3's `Pause` drops only the pause intent and `PauseCovers`. A shed state `paused` still derives `Paused`, so `Act` waits with `ReasonPaused`.
- So `lym batten cancel` refuses any run whose batten or child shed state is paused. Its way forward is `lym tender resume`, which does not clear a shed pause.
- That contradicts "ends a run for good, whatever its state" (`internal/battencli/cancelcmd.go:54`). `TestCancelRun` gains no paused-run row.

**MEDIUM — Card 14 (tender-pause-intent): `lym tender resume <slug>` refuses a batten-paused run by naming itself.**
- Its `Judge` sets only `Pause` and `Cancel` aside, not `ShedPause`.
- A shed `paused` record therefore waits, and the refusal names `lym tender resume <slug>` again, which never progresses.
- No `TestResume` row covers a run whose shed state is paused.

**MEDIUM — D3 criteria "the cancel intent is on origin", "a cancel interrupted on one machine reads cancelling on another", "pause intent on origin", "finish on origin": no row asserts origin or another machine.**
- Card 7's three cancel integration tests and card 14's tier1 test assert a commit on the records side only. The push is spawned detached.
- Nothing reads the intent from origin or from a second machine. `TestDerive`'s `Cancelling` row is records-only.

**MEDIUM — Card 5 (pairteardown-read-run): the "`.lym` removed reads the same state" row cannot fail.**
- `TestReadRun` runs over fakes. "Deletes every `.lym` file the fakes stand for" removes nothing the reader touches.
- A `ReadRun` that read a cache through a real path would still pass.
- D3's criterion needs a row over real files, at tier1, that deletes `.lym` and re-reads.

**MEDIUM — D1 "every reader of a run's state takes it from `internal/runfsm`" and "the local parent branch never decides it": not carried for the existing readers.**
- `hubChildReachedParent` (`internal/battencli/cancelcmd.go:383-397`, through `childParentHolds` and `fabricengine.ParentHoldsTaskWork`, local) still decides cancel's reached-parent refusal. It also decides `DobbyRuns`' `ReachedParent`.
- `tendercli` `EndOwnedElsewhere` (`internal/tendercli/deps.go:334-340`) also reads the local parent branch.
- The overview's state cells (`battencli` overview, `DobbyRunCells`) and `lym tender status` still read the shed state.
- No card changes any of them.

**MEDIUM — D3 "a session id whose transcript is absent starts that session fresh": only a doc sentence.**
- Card 15 changes `pullrequest.Record`'s doc only.
- No card changes or tests `PRReview`'s `resumeSession` (`landingshed/prsession.go:219`) or webster's state for a missing transcript.
- Moving the record into tracked records makes this path reachable on a second machine.

**MEDIUM — Card 16 (fixer-attempts-tracked): no row tells `RecordsDir` from `ScratchDir`.**
- `newTestDeps` and `NewTestDeps` set `RecordsDir` to the scratch directory.
- A member left on `ScratchDir` passes every existing fixer test.
- No row asserts that the attempts file lands under `loomengine.LoomDurableDir`, the directory `statusCommitPathspec` stages. The criterion "read on another machine after a pull" therefore has no row for the fixer counts.

**MEDIUM — Card 14: `hubRunIntents`' owner read has no row.**
- D3 says "the verb reads the owner from a Bolt it fetches first", with the local Bolt when origin is unreachable.
- Card 14 wires both branches through `boardengine.PullBolt` in `hubRunIntents`.
- Only the engine row with a fake `Owner` is planned. The tier1 test covers an owned-here slug's commit.

**MEDIUM — Card 8 (cancel-readers-intent): the way forward for a run cancelled on another machine is self-contradicting and untested.**
- The card says a run cancelled elsewhere reads as an interrupted cancel here, "the way forward names the cancel there".
- But `cancelMarkerRefusal`'s interrupted text stays "as today": run `lym batten cancel <slug>` from the prime, here.
- That cancel is refused on a machine that does not own the run (`cancelOwnedElsewhere`).
- No row covers an intent with a free local lock written by another machine.

**LOW — Card 9 (cancelled-record-cleanup): "pair is gone" is read through `PairParts.Any()`, which counts a leftover local task branch.**
- `Any` includes `TaskBranch` and `StrayPath` (`internal/fabricengine/pairparts.go:30-31`).
- A cancelled record whose worktrees are gone but whose local task branch stands is skipped forever, with "its pair stands; `lym fabric remove`".
- The record names the pair, meaning the worktrees.

**LOW — Card 3, with card 5: the "act for a held landing past its merge" branch cannot be reached in this part.**
- Card 5 reads `Held` from the adopt pause entry, which is also a covering entry.
- So `PauseCovers` is true whenever `Held` is, and step 4's `!PauseCovers` fails.
- `TestAct` reaches the branch only with synthetic inputs.

**LOW — rows missing for these branches:**
- Card 12: the `runVerdict` read error.
- Card 7: `battenPreStepUnmarked` returning `cancelledRunRefusal` (only the run path's `checkRunStatus` has a row).
- Card 14: `Record` called for an already-paused scope, and a slug with no batten record writing nothing.
- Card 6: `claimBoardEntry` still writes `running · Worktree-Create` rather than the FSM's `starting`, a display not written from the FSM state.

Tool calls:
- Read `fabricengine.PairParts` and `Any` (`internal/fabricengine/pairparts.go`), because card 9's "pair present" operand depends on which parts `Any` counts and my context did not hold it.
