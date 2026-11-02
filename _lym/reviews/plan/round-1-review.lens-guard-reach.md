```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: whether each guard, refusal, branch or test row a card adds can be reached.

**Per guard, reachable or not**

*Card 1*
- `SpawnOrAttach` refuses a role outside the fixer set: a fail-loud wiring guard, since every caller passes a constant. Exempt.
- Start, `FindRun`, `Register` and `Save` failures stop the strand: reachable through the seams.
- `Await` refuses an unknown engine or an unparsable `SpawnedAt`: reachable only through a corrupt or hand-edited record. Treated as I/O-class.
- `Registrar` refuses an absent strand or pane pid 0: reachable when a strand dies right after it starts.

*Card 2*
- `RenderPrompt` refuses an empty `ReplyPath` or `ReportPath`: a wiring guard, exempt.

*Card 3*
- A `Register` failure stops the strand, with nothing appended: reachable.

*Card 4*
- `fixer` on a key other than `card`: reachable from a hub's gate.yaml.
- An unparsable `fixer`: reachable.
- `fixer_timeout_min` below 1: reachable.

*Card 5*
- The loaders refuse an unparsable model-spec: reachable.

*Card 6*
- An unknown entry name: a wiring guard, exempt.
- `VisitDir` or store error: reachable, I/O-class.
- The `Attempts + 1` cap: reachable. A held re-prompt send clears the memo and re-evaluates on a later tick (`wait.go:546-558`, `wait.go:359`), and each such evaluation runs the fixer entry afresh.
- `SkipForked`: reachable.
- An `Await` error stops the strand and marks the record dead: reachable.
- The fixer entry's `Gate` on a live record (step 1): reachable only narrowly. After an attach, the `Expired` probe and the finalize through `Final` usually come first.
- `Expired` "false for one this wrap spawned": **not reachable** (finding 3).

*Card 7*
- A nil `Env.Setting`: a wiring guard, exempt.
- A missing setting, a non-string value, an unparsable value, a malformed `fixer` and `fixer` on two elements: all reachable through recipe or config edits.
- `fixer` on a row that does not read it: reachable, as the unknown-key refusal.

*Card 8*
- An alias the registry does not hold: reachable after `modelspec.Parse` passes.
- An entry outside the body table: a wiring guard, exempt.
- A missing status file in `rowVisitDir`: I/O-class.

*Card 9*
- Slug read error and `EndRole` failure: reachable.
- The end of the fixers itself fires on paths that do not move the run: **finding**.

*Card 10*
- `GateFixerSetting` refuses an unknown recipe: a wiring guard.

*Card 11*
- `strandRole` gives `""` for a name that does not parse: reachable.
- `callKeyDir` keys a fixer-role call apart: reachable. A `gate-fixer` strand resolves a positive keep-warm cap, either through its base role or the default fallback (`internal/shuttleengines/keepwarm.go:42`, `:60`), so its call is capped.

*Card 12*
- `handsOff` turning down fixer roles, a missing `--gate`, a key with no `fixer`, an empty slug and `card-verify`: all reachable.
- An empty request role "as an older front writes": **the named input cannot occur** (finding 4).
- Attach to a live record left by a runner that died, and the failed-stop exit 1: reachable.
- The lease lapsing during the fixer phase: reachable. Joining fronts extend the lease (`detachfront.go:587`), so it lapses only when the caller stops calling.

*Card 13*
- No guards.

**Findings**

1. **MEDIUM, design. Card 9 (`loomCLI.loomPreGoto`): the fixers are ended before the goto's own checks, so a goto that is then refused still kills the live fixer of a running row.**
   - `shedverbs.RunGoto` runs `PreGoto` first (`internal/shedverbs/goto.go:22-26`). Only after that does `shedengine.Goto` refuse a running run, an unknown target or a done run (`internal/shedengine/goto.go:96-110`).
   - So a mistaken `lym loom goto` on a running Plan-Write ends the fixer the row's closure is blocked on. The wrap's `Await` then reads the strand gone, the fixer counts as dead `died`, and the writer is re-prompted with a dead note.
   - The card only says its own refusals leave the run unmoved; it never bounds this.
   - Proposed fix: end the fixers in the told `GotoGuard`. `Goto` runs it under the run lock after its own checks and `CheckGotoTarget`, and before the write; `Relaunch` runs it the same way, and only for a non-empty `Target` (`goto.go:229-235`).

2. **MEDIUM, design. Card 9 (`loomCLI.goDriverDepsFor`'s `ConsumeTackler` and `loomCLI.loomConsumeAttempt`): `endRunFixers` runs before every consume, not only before a move to another row.**
   - It runs for a relaunch with an empty `Target`, which resumes the same row and writes the status unchanged (`goto.go:186-195`).
   - It also runs for a consume that `AdmitRelaunch` then refuses (`internal/sheddriver/tacklerconsume.go:50-56`, `internal/shedarbiter/consume.go:53-59`).
   - A resume of the same visit therefore kills the live fixer that D1 and D8 say a restarted caller attaches to. D8: "A crash re-step of the same row … keeps the same set, so the `Expired` probe and the wait-only entries see the live fixer."
   - Same fix as finding 1: end the fixers in the guard, which runs only for a non-empty `Target`.

3. **LOW, consistency. Card 6 (`gatefixer.WrapRowGate`'s `Expired`): the "false for one this wrap spawned and still waits on" branch, and its `TestWrapRowGate` row, guard a state that cannot occur when the probe runs.**
   - `probeExpiry` runs only from the wait loop (`internal/shuttleengine/wait.go:474-492`). That loop is blocked inside the closure while this wrap's fixer is live.
   - Every path out of the closure marks the record terminal first, except the failed-stop path. That path returns an error, which ends `Wait` (`wait.go:516-518`).
   - So the in-memory guid set never changes an answer. D2's own rule, "a live fixer record saved before the run's last attach", is simply "any live record" here.
   - Proposed fix: drop the in-memory guid set and that test row.

4. **LOW, consistency. Card 12 (`TestHandsOff` row and Intent, plus card 11's `detachRequest.Role` note): the named input "an empty request role, as an older front writes" cannot occur.**
   - The front launches the runner from `os.Executable()` (`internal/gatecli/detachfront.go:118-128`), so the front and the runner are always the same binary.
   - An empty role reaches `handsOff` only from a strand name that does not parse (card 11's `strandRole`).
   - Proposed fix: name the malformed-strand input in both cards and in the test row.

**Tool calls**
- `internal/shedverbs/goto.go:14-50`: the order of `PreGoto` against `shedengine.Goto`. My context had only the grep hit.
- grep of `internal/battencli` and `contracts/recipes`: whether batten has goto and consume paths that card 9 leaves unguarded. Batten has its own, but webster runs only in the tapestry recipe, so there is no finding there.
- `KeepWarm` in `internal/shuttleengines/keepwarm.go`: whether a `gate-fixer` strand's call is capped, so that card 11's keying can be reached.
- grep of `writeLease` in `internal/gatecli/detachfront.go`: whether joining fronts extend the lease during the fixer phase.
