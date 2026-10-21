# Support log — gate settings per machine

## Interview

### Turn 1 — exploration (autonomous; no operator)

Read the board entry, the related entries `bench-osl1046-xdr`, `loom-gates-xdr-stacking`, `gate-language-neutral`, `machine-handover` and `dotnet-bench-osl1046`, and the config layer machinery.
Findings that shaped the design:

- `gateslot.ConfigSpec` classifies every gate key `cross-pair`, which closes it to the local layer; the local layer is per worktree anchor (`<Anchor>/.lyx/config/<module>.local.yaml`), not per machine.
- The slot pool's locks live at `<BoardDir>/.lyx/gate/`, and `hubreconcile`'s stamp and lock at `<BoardDir>/.lyx/config/`: the board checkout's `.lyx` is already the per-machine, per-hub home.
- `hubgeom.GateSlots` reads the stack through `fabricengine.ConfigDirs` at every acquire, so a layer added in `configengine` reaches every gate site with no caller change.

### Turn 2 — design questions, all auto-picked

1. Where does a machine value live? Recommended and picked: a new `machine` layer at `<BoardDir>/.lyx/config/<module>.machine.yaml` (D1).
2. How is a key opened to it? Recommended and picked: a new `machine` category (D2).
3. Which keys? Picked: `slots` and `go_parallel`; `busy_quit_sec` stays hub-wide (D3).
4. Who writes it? Picked: `lyx config gate --layer machine`, prime-only, uncommitted (D4).
5. How does it stay current with template changes? First picked: the board-side reconcile prunes it; replaced in turn 3 by "never reconciled" (D5).

### Turn 3 — lens review

Every fork returned.
Duplicates are merged and named under each lens that raised them.

- walkthrough 1 (BLOCKING), existing-paths 1 (BLOCKING), interactions F3: hub-file checks load `Dirs{Board}` and would stack the machine file, so a bad hub value masked by a machine override passes and reaches Bolt — fixed: D1 applies the machine layer only when `Anchor` is also set; added an acceptance criterion.
- interactions F1, bound-breaker 3: a bad machine file reconciled in `Ensure` blocks preflight, arm, orch start and clone — fixed: D5 now never reconciles the machine file.
- interactions F2, walkthrough 3, existing-paths 2, bound-breaker 4: the board pass's commit and rollback cover only `_lyx/config/` — fixed by the same D5 change.
- interactions F5, bound-breaker 2: D5's removal of a pruned machine file added an unlisted exception to "Local files" — fixed: no removal; the record states the list is unchanged.
- bound-breaker 1: D5 both dropped and refused unknown keys — fixed: no reconcile, so a refused key fails the load only.
- interactions F4, walkthrough 2, existing-paths 3 and 4, bound-breaker 5: `GateSlots` and `lyx gate test` name only the hub file — fixed: D3 makes every gate load failure name both files and both ways forward, with removing the file named; added an acceptance criterion.
- bound-breaker 6: D4's `CheckLoad` stack description was inexact — fixed: says the prime's worktree stack.
- goal-scope 1: Goal read as fixing a sweep side effect — fixed: Goal names per-machine CPU tuning as the lasting purpose.
- goal-scope 2: generic mechanism beyond the entry — rejected: the layer model requires a layer and a category; nothing beyond gate is reclassified.
- goal-scope 3: no criterion for `go_parallel` reaching `GOFLAGS` — fixed: the first criterion checks the lease's `-p`.
- goal-scope 4, board-fit 1: `gate-language-neutral` moving `go_parallel` to `parallel` — fixed: Open risk added.
- goal-scope 5, leanness 2: Auto-mode assumptions restated decisions — fixed: kept the two assumptions not argued elsewhere.
- board-fit 2: note `xdr-git-test-file-burst` recommends `--layer local` for gate keys, refused before and after this task — rejected for the record: correcting another entry is the orch's call after landing; recorded in the question ledger.
- board-fit 3: the dev and prod risk wording — rejected: the risk holds whatever `deploy-prod-cleanup` changes; reworded to name builds, not `prod`.
- board-fit 4: a fresh clone or adopting machine starts without a machine file — fixed: named in the durable-vs-ephemeral constraint.
- leanness 1: D4 held a rejected alternative — fixed: the sentence is gone; D5 states the behaviour.
- leanness 3: the `PATTERN-tolerant-records` constraint bound nothing — fixed: cut.
- leanness 4: `busy_quit_sec` reason stated twice — fixed: reason kept in Scope Out only.
- leanness 5: the durable-vs-ephemeral PATTERN entry carried a path — fixed: entry change dropped; the path goes to its background file via Notes.
- leanness 6: the plan-writer note listed call sites — fixed: shortened to the refusal text.
- leanness 7: the Docs bullet listed every doc — fixed: only the two docs that state the hub-file-only claim are named.

## Rejected alternatives

- **Reclassify `go_parallel` to `CategoryNone` and use the existing worktree-local layer.**
  The local file is per worktree, so each pair would need its own copy, and two worktrees with different `slots` disagree on one machine-wide pool; a pair agent could also raise its own budget.
- **An environment variable (`LYX_GATE_GO_PARALLEL`) read by `lyx gate test`.**
  Not visible in `--resolved`, not validated by the loader, and lost in tmux strands that do not inherit the operator's shell; it also bypasses `PATTERN-config-layers`.
- **Per-machine sections on Bolt keyed by hostname (`machines: {osl-1046: {go_parallel: 8}}`).**
  Needs a machine identity that task `machine-handover` has not built, and still puts every machine's tuning on Bolt, which the entry wants to avoid.
- **A user-level file under the home directory, shared by every hub on the machine.**
  Hubs differ in language and suite size, so one value across hubs is wrong, and it adds a config root outside the hub geometry.
- **A machine layer for every module.**
  No per-worktree module asked for one; YAGNI.
- **A separate `MachineKeys` list on `Spec`.**
  Restates keys beside `Categories`; one classification per key is the existing model.
- **Including `busy_quit_sec`.**
  Not asked for and not measured to differ per machine; it gates a hang detector, so it stays cross-pair.
- **Reconciling the machine file in the board-side pass (`Ensure` and `--layer default`).**
  That pass commits every written file in `_board`, its rollback covers only `_lyx/config/`, and an `Ensure` failure blocks preflight, arm, orch start and clone; a file of at most two keys does not justify that.
- **A `lyx config reconcile --layer machine` verb.**
  No reconcile is needed (D5); the verb adds surface with no case for it.
- **The machine layer applying whenever `Board` is set.**
  Hub-file checks load `Dirs{Board}` alone, and would then let a machine override mask an invalid hub value.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Layer location — auto-pick: `<BoardDir>/.lyx/config/<module>.machine.yaml`; the board checkout is per machine and per hub, and the slot pool already lives there.
- Q2 Key opening mechanism — auto-pick: `CategoryMachine`; keeps one classification per key.
- Q3 Keys in scope — auto-pick: `slots` and `go_parallel`; the entry names `go_parallel` and "possibly `slots`", and the pool's machine-local locks make `slots` safe.
- Q4 `busy_quit_sec` — auto-pick: stays hub-wide; not asked for.
- Q5 Write guard — auto-pick: the default layer's prime-only guard; a machine value changes every run on the machine.
- Q6 Commit — auto-pick: never committed, no fabric sync; it must not reach Bolt.
- Q7 Reconcile — auto-pick, revised after the lens review: the machine file is never reconciled; loads drop a retired `machine` key with no warning, and any other retired or refused key fails the gate load with a message naming the file.
- Q9 Machine layer applicability — auto-pick: only for a hub-wide module with a `machine` key and only when both `Board` and `Anchor` are told, so hub-file checks stay machine-free, a stray machine file of another module is never read, and a worktree whose anchor has no `_lyx/` still stacks it.
- Q10 Note `xdr-git-test-file-burst` — open for the orch: it recommends `lyx config gate --layer local --set go_parallel=1` and `slots=1`, which is refused; after landing, the working command is `lyx config gate --layer machine --set …` from the prime. This run does not edit that entry.
- Q8 Per-worktree modules — auto-pick: no machine layer; a `machine` key there fails a registration test.
