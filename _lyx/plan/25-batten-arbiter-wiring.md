# Card 25 — batten-arbiter-wiring

**Edit:**
- `internal/battencli#battenCLI.driverDepsFor`
- `internal/battencli#battenCLI.loadedTacklerDeps`
- `internal/battencli#battenCLI.specFor`
- `internal/battencli#TestBattenDriverDeps`
- `internal/battencli#TestSpecFor_FillsRunIdentityForSlug`
- `internal/sheddriver/doc.go`
- `docs/overview.md`
- `PATTERN.md`

**Create:**
- `plan:internal/battencli#battenArbiterRecipe` -> `const battenArbiterRecipe = ""`
- `plan:internal/battencli#battenCLI.loadedArbiterDeps` -> `func (c *battenCLI) loadedArbiterDeps() (shedarbiter.Deps, error)`
- `plan:internal/battencli#battenCLI.battenConsumeAttempt` -> `func (c *battenCLI) battenConsumeAttempt(ctx context.Context, attempt string) error`
- `plan:internal/battencli#TestBattenConsumeAttempt` -> `func TestBattenConsumeAttempt(t *testing.T)`

**Uses:**
- `plan:internal/shedarbiter#Deps`
- `plan:internal/shedarbiter#Seam`
- `plan:internal/shedarbiter#ConsumeAttempt`
- `internal/sheddriver#ResolveArbiter`
- `plan:internal/arbiterstop#NewBattenReaders`
- `plan:internal/arbiterstop#BattenHandBackWayForward`

**Intent:**
Wire the arbiter into batten's Go driver, so every shed-driven recipe's stop chain is the rule table, the tackler, the arbiter and then the parent (D8).
`driverDepsFor` fills `Arbiter` with `shedarbiter.Seam` over `loadedArbiterDeps`, loaded on the seam's first call as the tackler's are: `sheddriver.ResolveArbiter(shed.yaml, batten.yaml's ShedOverride, registry)`, `arbiterstop.NewBattenReaders()`, the arbiter directory, attempt directories and marker under the prime's run records of the slug (`shedrun.ArbiterDir`, `ArbiterAttemptDir`, `ArbiterMarker`), the prime's shuttle runner that `loadedTacklerDeps` builds (factored so both share one runner build), batten's producers and a goto guard, the stencils dir, the slug and the prime's worktree, no decision record or support log (`none`), no amendment target, `battenArbiterRecipe` as the recipe, and `arbiterstop.BattenHandBackWayForward` as `HandBackWayForward`, so every refusal batten's arbiter call gives names `lyx batten status <slug>`; `EscalationPending` answers false, since a batten stop never awaits a review escalation.
Bound of the arbiter's goto: `shedengine.CheckGotoTarget` admits only rows at or before the current one, and within it a goto back would re-run Seed-Child, which rewrites the task worktree's local config layer and seed, or Worktree-Create, over a live pair; so `loadedArbiterDeps`' `GotoGuard` gives the arbiter the tackler's bound (`GotoRoutedFrom: false`) and refuses any target but the halted row, the status's current row read at the call, with `goto target <row> is not the halted row <halted>; batten's arbiter resumes only the halted row; way forward: the run's parent runs lyx batten goto <slug> --to <row>`.
`battenArbiterRecipe` describes batten's rows and what each does: Worktree-Create (claim and make the pair), Seed-Child (write the child's seed and run settings), Open-Window, Seed-IDE and Open-IDE (optional commands), Run-Shed (watch the child run), Land (pull the prime, deploy, prune the board) and Worktree-Teardown (end the session and remove the pair), and that the arbiter's only moves are a plain re-launch, a goto to the halted row, or a hand-on.
`OpenAttempts` names the newest arbiter attempt for the stop key beside the tackler's, so a restarted pass resumes an open arbiter attempt; `specFor` sets `Hooks.ConsumeAttempt` to `battenConsumeAttempt`, which parses the attempt number and calls `shedarbiter.ConsumeAttempt` over batten's relaunch request, so the driver's `--arbiter-attempt` relaunch consumes an arbiter resume.
`PATTERN.md` gains, in its Shed and loom section:

```
- `PATTERN-shed-stop-chain` — A shed run's stop: rule table, tackler, arbiter, then parent, for every recipe; arbiter keys in shed.yaml with module overrides.
```

`sheddriver/doc.go`'s seams paragraph and `docs/overview.md`'s batten paragraph say batten's driver wires the arbiter after the tackler, with `batten.yaml`'s arbiter overrides of `shed.yaml`.
Tests: `TestBattenDriverDeps` gains rows: the `Arbiter` seam is set and its `EscalationPending` answers false, and `OpenAttempts` names an arbiter attempt recorded at the key, and the arbiter deps' `GotoGuard` refuses a goto to Seed-Child from a stop at Land with the halted-row refusal and admits Land; `TestSpecFor_FillsRunIdentityForSlug` asserts `ConsumeAttempt` is set; `TestBattenConsumeAttempt` (tier0, hand-populated receiver, parallel) rows: `abc` and `0` are refused as no positive attempt number, naming the hand-back to the run's parent, before any attempt directory is read.
That a tackler escalation reaches `Judge` once the seam is set is the driver's own rule, pinned by `internal/sheddriver#TestGoDriver_Settle`'s escalate row, so batten's card pins only that its seam is set.

**ImpactSummary:** Batten's driver deps gain the arbiter seam with its halted-row goto guard, its open-attempts read and its loop's consume hook; loom's and darn's wiring is untouched.
