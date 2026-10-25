```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked each claim and bound in the record.

**Claims attacked**
- D7 step 1, "end the dobby process alone, leaving any pass it spawned to finish": **broken** (F1).
- D7 step 1 for the prime, `runproc.End` on the prime's owner ends "every dobby": **broken**, because it also ends the prime's records pushes (F1).
- D7 prime refusal "any live batten run", checked after step 1: **broken**, because step 1 has already killed those passes (F1).
- D7 step 7, "start again exactly the dobbies step 1 ended", on a re-run after a kill: **broken** (F3).
- D7 "the operator does not start one in the worktree meanwhile", the stated bound on there being no lock: **broken** by automatic dobby starts from hub-wide verbs (F5).
- D7 recovery claim that "reconcile does not repair it, since it treats a pair with no `_lyx` at its anchor as unmanaged": **broken** (F2).
- D7 "with no such trailer, as for an anchor inherited through a fork": **broken** (F4).
- D7 "A re-run first undoes an uncommitted move": **held, but underspecified** for the never-tracked `.lyx` (F8).
- D10 weft-pull refusal and its way forward: **broken for the prime** (F6).
- D10 "incoming tip carries a marker different from the pair's": **broken** while a pair's own anchor commit is unpushed (F7).
- D8 census bound, "removal waits until no live branch needs the fallback" while making no network call: **broken** (F9).
- D2 stale-spelling guard "applying to this read only", after D8 removes the Bolt marker: **weakened** (F10).
- D4 "the fresh-bind test counts a marker on `main-weft` or on Bolt": **unsupported** by the probe as it is built today (F11).
- D2 empty own marker counting as absent, and every `fabricengine` path refusing it: **held**; non-fabric commands fall back to `.` and the cwd gate refuses them loudly.
- D3 scan on `Location` literals: **held**.
- D6 re-run with a present pair and a different `--anchor`: **held**; it is refused and points to `fabric anchor --set`, and the re-run undoes the uncommitted move.
- D9 census keyed by anchored pattern: **held**.
- D10 checkout refusing before switching either side, and a forked new weft branch never being refused: **held**; the target's anchor can be read before the warp switch.
- On Windows, `runproc.End` signals nothing, and D7 refuses a live dobby: **held**.

**Findings**

**F1 — BLOCKING, design.** Location: D7 step 1, the "pass in flight finishes" acceptance criterion, and the prime refusals.
`runproc.End` cannot end "the dobby process alone".
- It ends every live process registered under the owner, and walks parent-pid descendants of each one.
- Dobby's passes are its children, started with `proc.Detach` but still descended from it by parent pid, so step 1 kills the passes it claims to leave running.
- For the prime, the owner is `runproc.HubOwner`, which also owns the prime's records pushes; those die too.
- The prime refusal "any live batten run" runs after step 1, so it never sees the runs it is meant to refuse: step 1 has ended them.

Evidence:
- `internal/runproc/end.go:52-58` (End ends "every live process registered under owner … and the processes below them")
- `internal/runproc/end.go:316-345` (the descendant walk by parent pid)
- `internal/beamcli/dobbydeps.go:461-479` (`spawnPass` is a child of dobby, `proc.Detach`)
- `internal/beamcli/dobbydeps.go:64-70` (dobby registered under the slug or `HubOwner`)
- `internal/fabricengine/spawn.go:128` (pushes registered under `HubOwner`)

The existing seam goes the other way: `EndSparing(…, RoleDobby)` spares dobby (`internal/pairteardown/skip.go:81`). There is no "end only this role" verb.

**F2 — MEDIUM, design.** Location: D7 recovery paragraph and plan-writer note 3.
The record's claim about reconcile is false for a pair whose weft is present.
- `isRawWarpWorktree` runs only on the missing-weft branch.
- A present weft goes through `repairPairWiring`, which wires the full junction set at whatever anchor the marker resolves to.

Counterexample 1: a kill after step 3's commit, then `lyx fabric reconcile`, which dobby or the operator may run. Reconcile wires junctions and the scratch link at B, but leaves the launchers at `_launchers/<A>` and the workspace files stale. The re-run must then cope with a half-done step 4.

Counterexample 2: a kill mid-step-3, before the commit. The on-disk marker still reads A, so reconcile re-points junctions at A into a weft whose content has already moved.

The record states neither interaction, and its rationale rests on a premise the code contradicts.

Evidence:
- `internal/fabricengine/reconcile.go:214-237` (a present weft gets `repairPairWiring`)
- `internal/fabricengine/reconcile.go:391-424`
- `internal/fabricengine/reconcile.go:514-522` (`isRaw` only after the missing-weft path)

**F3 — MEDIUM, design.** Location: D7 step 7 and the recovery paragraph; the acceptance criterion "a re-run after a kill at any step finishes at B".
Which dobbies step 1 ended is kept in no record.
- A re-run after a kill between steps 1 and 7 starts again "exactly the dobbies step 1 ended" of its own run, and its own step 1 finds none live, because the killed run already ended them.
- So they stay down. For the prime, that is every dobby on the machine.
- Recovery "from the record alone" cannot reach this state.

Evidence: D7 steps 1 and 7; `internal/dobbyengine/doc.go:12` (only `EnsureLive` restarts dobbies).

**F4 — MEDIUM, design.** Location: D7 recovery paragraph, and the acceptance criterion "a `--set` equal to an inherited anchor is a no-op".
"With no such trailer, as for an anchor inherited through a fork" is false.
- A pair forked after its parent's change A→B inherits the parent's history.
- So the commit that last changed its marker is the parent's change commit, which carries `Lyx-Anchor-From: A`.
- A re-run or `--set B` on that child reads the trailer and runs the A-side cleanup of steps 2 to 7 for a worktree that was never at A.
- That cleanup includes removing exclude lines and launchers, and ending and restarting dobby.

Evidence: D5 (forks carry the parent's marker through git); D7 recovery paragraph.

**F5 — MEDIUM, design.** Location: D7 paragraph "The verb holds no lock against a session or run started by hand".
That bound covers only hand starts in the worktree.
- `lyx orch start` and `lyx adopt`, run anywhere in the hub, call `dobbyengine.EnsureLive`.
- `EnsureLive` starts every owned worktree's dobby whose lock reads free.
- So a dobby step 1 ended is restarted mid-move, and its passes start at an anchor that is being moved.
- The record states no bound for these automatic starts.

Evidence:
- `internal/orchcli/start.go:271`
- `internal/adoptcli/deps.go:112`
- `internal/beamcli/cli.go:71`
- `internal/dobbyengine/doc.go:12`

**F6 — MEDIUM, design.** Location: D10 weft-pull refusal and D7 "never acts on another machine's worktree".
The prime is checked out on every machine and shares `main-weft`.
- Changing the prime's anchor on machine 1 commits and pushes marker B on `main-weft`.
- Machine 2's next prime weft pull carries a marker different from its own, so it is refused.
- Its way forward, `lyx fabric remove <slug>`, cannot be run on the prime: Remove refuses it outright.
- That violates PATTERN-refusal-way-forward, and D7 states no bound or path for a prime anchor change on a multi-machine hub.

Evidence:
- `internal/fabricengine/remove.go:131`
- `internal/fabricengine/remove.go:738`
- `internal/fabricengine/remove.go:942-949` (`refusePrimeSlug`)

**F7 — LOW, design.** Location: D10 pull refusal.
"Incoming tip" is undefined when the local branch is ahead.
- After D7 step 3 commits B, step 7's push is async and can fail.
- Read as "origin's tip", the refusal fires on every later pull while origin still holds A.
- Its way forward, remove and re-adopt, drops the unpushed anchor commit.
- The pull arm's replay path (`settleUnmovedWeft`), where both sides moved, also needs the marker compared before any update. The record does not say against which tip.

Evidence: `internal/fabricengine/pull.go:290-330`.

**F8 — LOW, design.** Location: D7 recovery, "undoes an uncommitted move".
`.lyx` is never tracked, so after a kill git shows no rename of `A/.lyx` to `B/.lyx`.
The record does not say how a re-run decides where `.lyx` sits, for example when both `A/.lyx` and `B/.lyx` exist or only one does.

Evidence: D7 step 3; PATTERN-durable-vs-ephemeral-state.

**F9 — MEDIUM, design.** Location: D8 census and the Open risks.
"A branch whose ref is not fetched keeps the Bolt marker" cannot be enforced without the network.
- A warp branch another machine pushed, which this machine never fetched, is invisible to the census, so it is not counted at all.
- Counterexample: machine 2, on the new binary, has a markerless pair whose branches were pushed but never fetched on machine 1.
- Machine 1 removes the Bolt marker.
- Machine 2's next board pull makes that pair resolve to `.` on a subpath hub, and every lyx command at its real anchor fails the cwd gate until reconcile runs there.
- The Open risks name only old binaries and local-only pairs; this case is neither.

Evidence: D8; `internal/lyxcwd/anchor.go:126-141` (the cwd gate).

**F10 — LOW, design.** Location: D2 step 2 and D8 Bolt removal.
- Today the stale-spelling guard fires only when `.lyx-anchor` is absent.
- Once D8 removes the Bolt `.lyx-anchor`, a leftover Bolt `.fabric-anchor` makes every markerless read return `ErrStaleAnchorMarker`.
- Clone's stale guard then tears down every re-clone of the migrated hub.
- D8 does not say to remove or refuse the stale spelling when it removes the marker.

Evidence:
- `internal/lyxcwd/anchor.go:158-167`
- `internal/fabricengine/clone.go:359-369`

**F11 — LOW, design.** Location: D4 fresh-bind test.
- `probeWeftBinding` makes a `--single-branch --depth 1` clone of the weft's HEAD, Bolt, and checks the marker only there.
- After D4, a new hub's marker lives on `main-weft`, which the probe never sees.
- Today this is masked because `Found` (`.lyx-warp`) is true on such hubs.
- D4's claim that the test counts a `main-weft` marker needs a probe change the record does not state.

Evidence:
- `internal/fabricengine/recordsprobe.go:51-53` (`freshBind`)
- `internal/fabricengine/recordsprobe.go:74-77` (the single-branch HEAD clone)
- `internal/fabricengine/recordsprobe.go:108`

**Tool calls**
1. Grepped the `EnsureLive` callers and `remove.go`'s prime handling, to find what restarts a dobby and whether the prime can be removed; the manifest held neither.
2. Grepped for raddle writers at landing, to test whether a child's Finalize writes into the parent's anchor; this was not confirmed, so no finding was raised.
3. Grepped landing and batten packages for writes at the parent's anchor, for the same check, with the same result.
