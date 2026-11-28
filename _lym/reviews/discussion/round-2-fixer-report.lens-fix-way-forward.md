```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether each way forward the fix added can be carried out from the state it names, and whether it leads somewhere that makes progress.

1. **MEDIUM.** Changed text: D4's jacquard bullet, case "any other pair, from its task worktree: … then `lym loom goto --to Discussion-Write` and `lym loom start`".
   Summary: the sequence ends with `lym loom start` over a run that goto has just left paused, and nothing shows that `start` lifts a goto pause.
   Evidence:
   - goto's help (`internal/loomcli/cli.go:498-516`) says goto "leaves it paused" and records itself as the run's pause reason.
   - The sibling tapestry bullet sends the same goto on with `lym loom resume`.
   - In `internal/loomcli/start.go`, `runHaltedAtHandBack` (`:142-158`) and `runDriverSpawnAndWait` (`:281-293`) resume a parked driver or spawn one. Neither has a code path that clears a recorded pause: a grep of start.go and resume.go for `StatePaused`, pause clearing and pause reasons found only the hand-back state list at `:154`.
   - `start` is still needed, because only its bootstrap commits the re-seeded seed (`sharedbootstrap.go:158-163`).

   So the run may stay paused at Discussion-Write after `start`. The sequence should either end with `lym loom start`, then `lym loom resume`, or cite the code that shows `start` lifts a goto pause.
2. **LOW.** Changed text: D4's same case, "remove its seed file and re-seed it with `lym shed seed self --recipe tapestry`, keeping the seed's driver and parent".
   Summary: the Stuck reason should give the seed's path and the full re-seed command, as `disagreeingSeedRefusal` does (`sharedbootstrap.go:254-257`).
   Evidence: an operator who reads only the Stuck reason cannot otherwise find the seed's path or its recorded driver and parent. The record leaves this implied by "as `disagreeingSeedRefusal` does".
3. **LOW.** Changed text: D5, "its teardown deletes the branch locally and on origin, and the start that follows the clear meets no leftover branch".
   Summary: when Worktree-Teardown's level read is false, it ends Done and keeps the pair, so the start after `lym batten cleanup --apply` can still meet a leftover branch.
   Evidence: `internal/battenshed/teardown.go:97` and `:131-133`.
   The start still makes progress: Worktree-Create's refusal names `lym batten cancel &lt;slug&gt; --remove` and then `lym batten run &lt;slug&gt;` (`internal/battencli/wire.go:371`). The sentence overstates the outcome, but the way forward is not broken.

No other way forward the fix added is blocked:
- Re-seeding with tapestry keeps Record-Seed as a row, since D2 keeps row names.
- goto moves only back, and Discussion-Write comes before Record-Seed in the expanded tapestry recipe.
- After Discussion, Record-Seed finds the record and writes nothing.

Tool calls beyond my context:
- grep for the goto implementation and start's park handling: my context lacked where goto and park/resume are implemented.
- sed of the `start.go` park helpers, `drivewiring.go` `driverGoto` and `shedverbs/goto.go`: to check what state goto leaves and how `start` treats a halted run.
- grep of `start.go` and `resume.go` for pause clearing: to check whether `start` lifts a goto pause.
