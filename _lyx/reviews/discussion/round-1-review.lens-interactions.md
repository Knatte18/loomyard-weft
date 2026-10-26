```yaml
lens: interactions
outcome: reported
```

Task: interactions lens — check every pair of decisions acting on shared state, plus each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**

- D1 beam start × D9 Beam-Await stops dobby on accept: held (D3 refuses a start once the run is past Beam-Await).
- D4 bound (a hand `lyx batten run` restarts the dobby) × D9 (the dobby is stopped so it starts no new member): held. A restarted dobby's member starts are refused at Worktree-Create until Beam-Down stops the dobby again.
- D8 Beam-Up removes the land request × D9 request bound to the seed id: held, the two are redundant.
- D8 Beam-Up runs the beam steps × D3 refusals: held, since Beam-Up never takes the `lyx beam start` refusal path.
- D7 watcher stops the orch × D9 failure posted to the beam's orch: held, since the owner yield ends the wait first.
- D7 × D6 adopt runs the beam steps on the new machine: held.
- D7 × D1/D3 start on the old machine: held (D3 refuses an owned-elsewhere beam).
- D5 `Effective` × D11 pawl owner verdict × D6 "a beam's own pawl run is resumed only by adopting its parent" × D10 run driven by the parent's dobby: **broken** (F1).
- AC "adopt G … does not resume G's own pawl run while another machine owns the parent" × D6: **broken** (F2).
- D6 adopt runs the beam steps × D3 "past Beam-Await" refusal: **broken** (F3).
- D3 "past Beam-Await" × D9 "while at Beam-Await" × D11 Worktree-Create hold: **broken** (F4).
- D5 drops task owners × D6 `--all`, refusals and upgrade bullet: held.
- D9 owner yield × D9 InnerRun notice suppression: held, with the re-spawn bound stated.
- D9 accept stops the dobby × D12 can-land notice from the dobby: held.
- D10 land checks × D9 Beam-Await re-running them on the parent's machine: held, given the entry's own-the-parent rule.
- D10 request in the beam's `.lyx/beam/` × Beam-Await driver in the beam pair: held while one machine holds both.
- D11 cancel's slug-wide end × D4 role-scoped dobby end: held, since cancel means to end both.
- D4 `beam stop` × D8 Beam-Down × D9 dobby already stopped: held (idempotent).
- D2 resume × D8 Beam-Up orch start, and D2 × D4 `orch stop` keeping the session id: held.
- D1 `--all` × D3 refusal of a beam past Beam-Await: held, one entry fails and the others go on.
- D5 claim refusal × D11 pawl owner verdict: held.
- D10 create claims root and records the beam owner × D5 "no root owner reads a recorded beam owner as recorded": held.
- D9 entry editable at Beam-Await × D5/D10 owner-setter beam write: held. `scopeChanged` compares `Beam` but the write comes before the claim, and `OwnerMachine` is never compared.
- D9 snapshot at accept × D8 perch answer key: held.
- Constraints: config-layers × D1/D6/D8 held; run-process-owner × D4/D11 held; tolerant-records × D9 held; declared PATTERN-owner-machine × F1 broken.
- AC 216, 225, 227, 233 × D4, D9, D10, D12: held.

**Findings**

**F1 — BLOCKING, design. D6 last bullets × D11 "A pawl run's owner verdict reads the beam project's own owner" × D10 "driven on by the parent's dobby".**
A pawl run can be left with no driver on either machine, and nothing names a way forward.

Counterexample: root and G are both owned by Neve. OSL-1046 runs `lyx adopt root`.
1. G's pawl run reads as owned by Neve, so D6 does not resume it on OSL.
2. Neve's prime dobby exits because root is owned elsewhere (`internal/dobbyengine/loop.go:208`). No batten pass on Neve owner-pauses or drives the run.
3. OSL then runs `lyx adopt G`. D6 resumes only runs whose parent is an adopted worktree, and G's pawl run's parent is root, so it is again not resumed. D11 says dobby never starts a beam project.
4. The run still reads "running at Beam-Await", so `lyx beam land G` on OSL passes D10's checks. It writes a request that no Beam-Await driver on OSL consumes.

The order in the entry's example (adopt G, then root) works; the reverse order strands the run. The fix is to resume a beam's pawl run when adopt makes this machine own both the beam and its parent, whichever is adopted last.
Evidence: decision-record.md:97-98, :144, :162-163; `internal/dobbyengine/loop.go:204-210`.

**F2 — MEDIUM, consistency. Acceptance criteria line 220 × D6 line 97.**
The criterion says adopting G "does not resume G's own pawl run while another machine owns the parent". That implies it does resume the run when the parent is owned here. D6 says a beam's pawl run "is resumed only by adopting its parent", whoever owns the parent. The two disagree on the case where both are owned here, which is the case F1 needs.
Evidence: decision-record.md:97, :220.

**F3 — MEDIUM, design. D6 "Adopt then runs each adopted project's beam steps (D1)" × D3's refusal of a beam past Beam-Await.**
D1 defines the beam steps as the dobby start, the window and the orch start, without D3's refusals.
Counterexample: Neve's pawl run of G is past Beam-Await (Parent-Align, the perch, Finalize). `lyx adopt G` on OSL adds G's pair and starts G's dobby and orch in a beam whose landing is in flight. Beam-Down then runs on Neve and stops only Neve's parts, so OSL keeps a live orch and dobby on a landed beam.
D6 states no bound or refusal for adopting a beam past Beam-Await.
Evidence: decision-record.md:36, :56, :99, :137-138.

**F4 — MEDIUM, consistency. D3 last bullet and D9 last sentence ("past Beam-Await") × D9 "While the pawl run is at Beam-Await, its run status is no landing hold".**
The two define the hold over different intervals. The rows before Beam-Await (Worktree-Create, Beam-Up, a halt at Beam-Up) are neither "at" nor "past" it.
- D3 lets `lyx beam start` run there.
- D9's rule, read as written, keeps the run lock, the Running layer and `ResolveBattenPrime`'s refusal in force there, because the board run status, written as "<state> · <producer>", names Beam-Up rather than Beam-Await.
- D11 relies on the hold covering Worktree-Create.

So a pawl run halted at Beam-Up leaves a beam whose orch is up (Beam-Up starts it) but that no batten verb can run from. The fix is to state the hold once, over named rows of the pawl recipe, and have D3, D9 and D11 refer to it.
Evidence: decision-record.md:56, :137-138, :161; `internal/loomcli/wiring.go:211-216`; `internal/battencli/refusal.go:95-104`.

Tool calls: none.
