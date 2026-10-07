# Card 5 — rerun-removes-superseded-strand

**Edit:**
- `internal/shuttleengine/attach.go`
- `internal/shuttleengine/attach_test.go`
- `internal/shuttleengine/doc.go`
- `internal/shedadapters/bouncer.go`
- `internal/shedadapters/singlellm.go`
- `internal/testkit/shedfake/shedfake.go`
- `internal/loomcli/smoke_attachprobe_test.go`
- `contracts/specs/refusal-spec.md`

**Intent:** D4: when a producer row starts a fresh shuttle run after its attach probe found nothing attachable, shuttle first removes the live strand of every earlier run of the same output-file set whose run the probe judged respawn-eligible.
The attach probe already decides "same producer, same outputs", so extending its respawn branch closes the gap for every shuttle-backed row that starts a run after a not-found probe (SingleLLM, Burler's `probeLiveRound`, the Bouncer's seed and judge starts, the reflection run) with one rule, and re-running a producer is the operator's decision to supersede the halted session.

In `Runner.AttachGated`, keep the respawn-eligible candidates the disposition loop already computes.
On the final `len(attachable) == 0` return, and only there (after the errored and multiple-attachable refusals), for each respawn-eligible candidate whose strand `strandStatusByGUID` reports tracked and live, call `r.reed.RemoveStrand(guid, false)` and log the teardown at Warn through `logger`, naming the strand guid and the run directory, per `PATTERN-spawn-observability`.
A removal failure returns an error and no "nothing to attach" answer, so the caller spawns nothing beside the live strand: `shuttle: attach: could not remove the superseded strand <guid> of run <runDir>: <cause>; way forward: run "lyx reed remove <guid>", then re-step the row`.
`Attach` and `AttachGated` are the entries a caller uses when it starts a run on a not-found answer, so both carry the teardown.

Add `func (r *Runner) AttachIfLive(spec Spec) (Result, bool, error)` in `attach.go`, the same probe with the teardown switched off, for a caller that starts nothing after a not-found answer: both entries share one unexported body taking a flag for the teardown, and the flag is false only here.
The two Bouncer probes that only wait on a live run, `awaitLiveJudge` and `awaitLiveSeed` in `internal/shedadapters/bouncer.go`, move to it, so a probe that starts no run removes nothing.
The `Shuttle` seam interface in `internal/shedadapters/singlellm.go`, which the Bouncer drives, and `internal/testkit/shedfake`'s `Shuttle` gain `AttachIfLive`; the fake answers as its `Attach` does and records no removal.

Update `AttachGated`'s doc comment, the file comment of `attach.go` and the `Runner.Attach` paragraph of `doc.go` to state the removal, its bound and the probe-only form.

Bound: it removes only strands whose run record declares the exact output-file set of the new run and whose run the probe judged respawn-eligible, so it never touches another producer's strand, a parallel same-role strand with other outputs, or a run the probe would attach to; a probe that refuses (errored or multiple attachable candidates) or attaches removes nothing, the zero-candidate and reed-state-gate paths have no strand to remove, and `AttachIfLive` removes nothing on any path.
Removing strands, never worktrees, keeps `PATTERN-pair-teardown` and `PATTERN-batten-bookend` untouched.

Add a row to `contracts/specs/refusal-spec.md`'s loom section, per `PATTERN-refusal-way-forward` (the error reaches `lyx loom step` and `run` as a producer failure): `producer re-run: superseded strand not removed`, transient, way forward `run "lyx reed remove <guid>", then re-step the row`.

Tests:

- `attach_test.go`: extend `TestAttach_OutcomeDisposition` (or add one table-driven test beside it) with rows over the fake reed, through plain `Attach`, since `AttachGated` shares its body: a terminal-outcome candidate on a live tracked strand is removed and Attach answers not found; a dead-pane candidate and an untracked one remove nothing; a candidate with another output-file set is never matched and its strand survives; an attachable candidate is attached to, never removed; a `RemoveStrandErr` makes Attach return the error naming the guid and `lyx reed remove`, with no not-found answer.
  The same terminal-outcome row run through `AttachIfLive` answers not found and removes nothing.
- `internal/loomcli/smoke_attachprobe_test.go` (tmux tier, the CLI-level check): beside `TestSmokeBurlerRound_AttachesToALiveRoundInsteadOfRespawning`, a test that leaves a halted round's run record at a terminal outcome with its shell strand still live, re-runs the producer through the same real `Runner`, and asserts the earlier strand is gone from `lyx reed status` before the new strand is added, while a live strand with other outputs survives.

**ImpactSummary:** `Attach` and `AttachGated` not-found returns now tear down live respawn-eligible strands of the same output set, reaching every caller that starts a run on that answer; the two Bouncer wait-only probes move to the new `AttachIfLive` and keep a pure probe, and attach or refusal paths are unchanged.
