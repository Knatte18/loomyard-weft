```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked the decision pairs, acceptance criteria (AC) and constraints below.

**Pairs checked**
- D7 recovery trailer rule × D7 step 3 commit, kill before the commit: **broken** (F1)
- D7 recovery × D6/D7 inherited trailer, and the AC "`--set` equal to an inherited anchor is a no-op": **broken** (F2)
- D7 step 1 dobby end × D7 step 7 restart × recovery re-run: **broken** (F3)
- D7 × D10 pull refusal on another machine's prime: **broken** (F4)
- D10 pull "incoming tip" × D7 step 7 push that failed: **broken** (F5)
- D8 census × D2 empty-marker rule: **broken** (F6)
- D2 "before touching anything" × D7 step 1 order, and the empty-marker AC: **broken** (F7)
- Constraint `runproc.End` × D7 step 1 "dobby process alone", and the AC "a pass in flight finishes": **broken** (F8)
- D7 refusal set × child runs that the worktree's dobby drives and that land into it: **broken** (F9)
- D8 Bolt removal × D2/D4 stale-spelling guard: **broken, LOW** (F10)
- D6 re-run "skip" × D7 kill between commit and wiring: **held, but underspecified** (F11)
- D5 Add anchor × D3 constructor on the not-yet-written weft: held
- D5 markerless value × D2 read for a markerless pair while Bolt still has a marker: held
- D5 markerless value × D10 checkout comparison: held
- D9 census × D7 steps 2 and 4: held
- D3 portal sweep × D7 prime re-anchor: held, bound stated
- D4 adopt × D8 Bolt removal: held
- D10 checkout fork case × D7: held
- D10 promote × D7 way forward: held
- D8 migration × D10 pull of main-weft across machines (identical marker adds): held
- Constraint "git only in `fabricengine`, checks in the CLI layer" × D7 and D10 work placement: held
- AC "child lands into parent, marker unchanged" × D3 target Location: held

**Findings**

F1 — BLOCKING, design. D7 recovery paragraph.
- A re-run after a kill before the step-3 commit takes `A` from the wrong trailer.
- That kill leaves no commit of its own, so "the commit that last changed the marker" is an older change or an inherited one, or there is none.
- The re-run then runs steps 2 to 7 against that older anchor, or against nothing. Step 3 moves content from the wrong source and writes marker `B`, while the restored content sits at the current `A`.
- This contradicts the AC "a re-run after a kill at any step finishes at B".
- The source anchor must come from the checked-out marker when the last marker commit is not this change's.
- Evidence: `_lyx/discussion/decision-record.md:126-128`, `:199`.

F2 — MEDIUM, design. D7 recovery against D6 and the ACs.
- The record says an inherited anchor has no trailer. That is false when the parent got its marker through D6 or D7.
- A child forked after such a change inherits the tip commit, `Lyx-Anchor-From: A` included.
- A `--set` equal to the child's anchor then reads as an unfinished change rather than a no-op: it runs steps 2 to 7, spawns a push and restarts dobbies.
- "Settled" is not defined anywhere.
- Evidence: `decision-record.md:89`, `:118`, `:128-129`, `:199`.

F3 — MEDIUM, design. D7 steps 1 and 7 against recovery.
- The set of dobbies step 1 ended lives only in the killed process.
- A re-run's step 1 ends nothing, so step 7's "exactly the dobbies step 1 ended" restarts nothing. A dobby the first run ended stays down after a run that otherwise recovers.
- Evidence: `decision-record.md:114`, `:124`, `:128`.

F4 — BLOCKING, design. D7 prime re-anchor against the D10 pull refusal.
- D7 lets this machine re-anchor the prime and push `main-weft` with marker `B`.
- Every other machine's prime then pulls an incoming marker that differs from its own, and D10 refuses with "`lyx fabric remove <slug>` and adopt again". The prime cannot be removed, so the way forward does not exist (`PATTERN-refusal-way-forward`).
- The record names no way for other machines' primes to follow.
- Evidence: `decision-record.md:98-99`, `:110`, `:159`.

F5 — MEDIUM, consistency. D10 pull against D7 step 7.
- "Incoming tip" is undefined when the local branch is ahead.
- After a re-anchor whose push failed, the upstream tip carries `A` and the pair carries `B`. If the upstream ref counts as incoming, the pull is refused.
- Its way forward, remove and re-adopt "at the incoming anchor", would throw away the local unpushed move.
- Evidence: `decision-record.md:124`, `:159`; `internal/fabricengine/pull.go:290-326`.

F6 — MEDIUM, consistency. D8 census against D2.
- D8 does not say whether a present-but-empty committed marker counts as "carrying a committed marker".
- If it counts, the Bolt marker is removed, and that pair then resolves to `"."` through D2: empty counts as absent, and Bolt is gone.
- The AC "every counted branch marked" leaves this open as well.
- Evidence: `decision-record.md:49`, `:140`, `:143`, `:202`.

F7 — LOW, consistency. D2 against D7's ordering.
- D2 says every `fabricengine` path refuses an empty marker "before touching anything", and the AC says the anchor verb's refusal touches nothing.
- D7 step 1 ends the dobby, or every dobby for the prime, before the empty-marker refusal runs.
- Evidence: `decision-record.md:50`, `:106`, `:114-115`, `:192`.

F8 — BLOCKING, design (likely duplicated by bound-breaker or existing-paths). The `runproc.End` constraint against D7 step 1 and its AC.
- `runproc.End` ends every record under the owner and walks the ppid descendants of each. Dobby's passes are its children: `spawnPass` uses `proc.Detach` but keeps the parent link.
- So "the dobby process alone, leaving any pass it spawned to finish" cannot be done with `End`.
- For the prime, `HubOwner` also covers the prime's records pushes, which `End` ends too.
- The AC "a pass that dobby had in flight finishes" contradicts the constraint.
- Evidence: `internal/runproc/end.go:52-56`, `:316-345`; `internal/beamcli/dobbydeps.go:64-70`, `:462-481`; `internal/fabricengine/spawn.go:128`; `decision-record.md:114`, `:170`, `:197`.

F9 — MEDIUM, design. D7 refusals against step 1's left-running passes.
- For a non-prime worktree, D7 refuses only runs "of the worktree".
- The passes its dobby drives are child runs whose Finalize lands into this worktree, under `parentlock`, with merge geometry from this worktree's anchor.
- D7 neither refuses a held parent lock nor states what such a landing can do while steps 2 to 4 remove and rewire the junctions. The "no lock" bound covers only hand-started work.
- Evidence: `decision-record.md:108`, `:114`, `:131`; `internal/dobbyengine/doc.go` passes section; `internal/fabricengine/mergepaths.go:26-31`; PATTERN-parent-lock.

F10 — LOW, consistency. D8 Bolt removal against the stale-spelling guards.
- D8 removes only `.lyx-anchor` from Bolt.
- A Bolt that also carries `.fabric-anchor` then trips `ErrStaleAnchorMarker` for every markerless read. Clone's own stale check on the board dir, which D4 does not limit to the fallback, also refuses a re-clone even though `main-weft` has the marker.
- Evidence: `internal/lyxcwd/anchor.go:161-166`; `internal/fabricengine/clone.go:359-369`; `decision-record.md:45`, `:67`, `:140`.

F11 — LOW, consistency. D6 re-run.
- After a kill between D6's marker commit and the wiring at `<dir>`, a re-run sees an equal anchor and "skips".
- The record does not say whether that skip still wires the junctions, scratch link and launchers at `<dir>`. D7's recovery is not named for this case either.
- Evidence: `decision-record.md:89-90`.

**Tool calls**
1. A grep for raddle writers, to check whether landing writes into the parent's `_lyx`. The manifest had no landing-side reads.
2. A grep for raddle in `merge_verbs.go`, `doc.go` and `weftwiring.go`, the follow-up to call 1. It was inconclusive.
3. `mergepaths.go` `resolveMergeGeometry`, to confirm landing geometry uses the parent Location's anchor. The manifest did not cover it.
