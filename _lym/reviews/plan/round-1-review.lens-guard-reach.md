```yaml
lens: guard-reach
outcome: reported
```

The guard-reach lens: whether each guard, refusal, branch or test row a card adds can be reached from a real entry point.

**Per-guard list**
- Card 4, `Registry.Check` rule 1 (empty id): a fail-loud wiring guard, exempt from the reach rule.
- Card 4, `Check` rule 2 (copied file): reachable once the copied file's source machine has renamed, at a catch-up or owner write.
- Card 4, `Check` rules 3 and 4 (name clashes): reachable at a catch-up, an owner write and `lym machine name`.
- Card 4, `EnsureIdentity` refusal (no config directory): reachable from `lym beam start` through `DobbyStart` (card 8), from owner writes (card 6) and from the pass catch-up (card 20).
- Card 5, `WriteHold` kind and `FixTask` refusals: wiring guards, exempt.
- Card 5, another machine's standing hold, and the same hold written twice: reachable.
- Card 6, `Machine` ≠ `Identity.Name` refusal: a wiring guard, exempt. Every production owner write is covered: `beamcli/deps.go:64`, `hubgeom/dobbygeom.go:108`, `orchcli/ownerpause.go:138`, `adoptcli/deps.go:92`, `battencli/owner.go:61`, and machinecli's sweep through card 10.
- Card 6, owner write refused by a registry clash: reachable.
- Card 10, refused new name, refused current or hostname name, copied id: reachable.
- Card 11, `--forget-former` of a name the file does not hold: reachable.
- Card 12, `CoveredError` and `heldLine` worded for `bolt-hold`: reachable through `loomcli/pausehold.go:57`, `shuttleengine/open.go:77` and the pause hold wait.
- Card 13, held refusal and the empty-`HoldMachine` wording: reachable. The empty wording comes only from hand-built inputs, as the card says.
- Card 14, `ReadRunPulled` falling back on unreachable origin and refusing on a dirty or diverged Bolt: reachable. No hub fixture sets `BOARD_SKIP_*`, so `PullBolt` really pulls in the tier1 test.
- Card 15, `SyncBoltHold` write and drop branches: reachable.
- Card 15, `Resume`'s refusal of a slug only a `bolt-hold` entry covers, and its write of a `bolt-hold` entry when a slug's own entry is lifted under a hold: possibly unreachable from the verb (finding 2).
- Card 16, `startReady` skipping a held task: reachable.
- Card 16, the held refusal in `refuseStartOwnedElsewhere`: possibly shadowed by an earlier check (finding 3).
- Card 17, `ErrHandedOver` answered as nil, the transient push error, and the takeover-first row: reachable.
- Card 18, unreachable origin, registration refusal, per-run failure joined: reachable. A nil seam is a wiring guard.
- Card 19, a slug with a live entry skipped; `RecordPauseIntent`'s owed push returned as an error: reachable.
- Card 20, catch-up refusals (exit `PassExitOriginWait`) and the hand-over exit: reachable. The pass runs `preflightRunOwner` before it takes any lock (`battencli/arm.go:746`, `cli.go:575`; the run-lock probe at `arm.go:927` is released at once), so the hand-over's local pair removal is not blocked by the pass itself.
- Card 21, catch-up owed at a fresh start, a sleep, and exits 80 and 77: reachable.
- Card 21, catch-up owed when a listed run answers `hand_over`: reachable, and it recurs without end (finding 1).
- Card 21, the `refused` notice for an unresolved config directory: reachable only on a narrow path (finding 4).
- Card 21, `syncBoltHold`'s held list (a `push_owed` run left out, a covered run): reachable. Under a hold, `held` comes before `pause_covers` in the act order (`runfsm/act.go:117-120`).
- Card 22, another machine's standing hold, a cut first read, and a push that hands the run over: reachable.
- Card 23, the pre-push stop and the InnerRun hold now rely on card 21's `hand_over` trigger: reachable.

**Findings**

1. **BLOCKING, design. Card 21 (`daemon.drivePasses`, the `hand_over` trigger).** The catch-up is owed again every tick for as long as any listed run reads as owned by another machine.
   - The dobby lists runs from the prime's batten records (`shedrun.List(prime)`), filtered only by parent worktree, with no check that the pair stands here (`internal/battencli/dobbyreads.go:77-107`, `158-200`).
   - So a run another machine owns, whether this machine already handed it over or never had its pair, answers `hand_over` on every tick.
   - Each tick then re-owes the catch-up and runs it again: a Bolt pull, a pull-first registration commit and the pair fetches, without end. Card 21 also holds passes while the catch-up is owed.
   - The trigger needs a bound, such as "pair present here", or no hand-over mark yet for the slug.

2. **MEDIUM, design. Card 15 (`tenderengine#Resume`).** The new `bolt-hold` resume rules may be unreachable from `lym tender resume <slug>`.
   - The verb hands `Resume` the act-rule verdict (`internal/tendercli/cli.go:246-254`).
   - That verdict uses `resumeAside`, which sets no `Hold` (`internal/tendercli/runintents.go:31`), and card 13 makes `Held` true for every slug under a hold.
   - If `Resume` refuses on a non-act verdict before its `bolt-hold` rules run, card 15's refusal, and its "lift the slug's own entry and write a `bolt-hold` entry in its place", are reached only by `TestResume`'s fakes.
   - Card 15 states no order between the verdict and its rules. I did not read `Resume`'s use of `Verdict`, so this needs confirming.

3. **MEDIUM, consistency. Card 16 (`battencli#battenCLI.refuseStartOwnedElsewhere`).** The new held refusal may be shadowed by an earlier check.
   - `startFirstPass` runs `refuseByAct` before `preflightRunOwner`, which leads to `refuseStartOwnedElsewhere` (`internal/battencli/cli.go:570-575`).
   - If `refuseByAct` judges through `ReadRun`, card 13's hub-wide `Held` refuses first with runfsm's held wording, so card 16's refusal is reached only by `TestRefuseStartOwnedElsewhere`'s direct call.
   - I did not read `refuseByAct`'s body, so this needs confirming.

4. **LOW, design. Card 21 (`daemon.catchUp`, the `refused` wait).** The notice text for an unresolved config directory is reachable only in a narrow window.
   - With the variable unset, `DobbyStart`'s identity read refuses at `EnsureLive` (card 8), so no dobby starts.
   - A running dobby keeps the environment it started with, so it cannot later lose the variable.
   - The branch is therefore reached only by a dobby started before the upgrade and re-exec'd onto the new binary, plus the fake in `TestRun_CatchUp`.

**Tool calls**
1. `dobbyParentRuns` (`dobbyreads.go:77-107`): how the dobby lists runs; the manifest had only `DobbyRuns`' body.
2. A grep for `BOARD_SKIP` in hubforge, testkit and gitkit: whether hub fixtures skip `PullBolt`, for card 14's rows.
3. A grep of `tendercli/cli.go` for `Verdict`, `Refusal` and `Resume`: the order of the resume verb's verdict; the verb's file was not read.
4. A grep for production `SetOwners(` callers: whether card 6's identity check covers every owner write.
5. A grep for the lock calls and `preflightRunOwner` in battencli: when a pass takes its locks relative to card 20's catch-up.
6. `arm.go:736-750`, `arm.go:915-935` and `cli.go:560-580`: the pre-run order around the owner preflight and the run lock.
