# Card 5 — rerun-removes-superseded-strand

**Edit:**
- `internal/shuttleengine/attach.go`
- `internal/shuttleengine/attach_test.go`
- `internal/loomcli/smoke_attachprobe_test.go`
- `contracts/specs/refusal-spec.md`

**Intent:** D4: when a producer row starts a fresh shuttle run after its attach probe found nothing attachable, shuttle first removes the live strand of every earlier run of the same output-file set whose run the probe judged respawn-eligible.
The attach probe already decides "same producer, same outputs", so extending its respawn branch closes the gap for every shuttle-backed row (SingleLLM, Burler's `probeLiveRound`, the Bouncer) with one rule, and re-running a producer is the operator's decision to supersede the halted session.

In `Runner.AttachGated`, keep the respawn-eligible candidates the disposition loop already computes.
On the final `len(attachable) == 0` return, and only there (after the errored and multiple-attachable refusals), for each respawn-eligible candidate whose strand `strandStatusByGUID` reports tracked and live, call `r.reed.RemoveStrand(guid, false)` and log the teardown at Warn through `logger`, naming the strand guid and the run directory, per `PATTERN-spawn-observability`.
A removal failure returns an error and no "nothing to attach" answer, so the caller spawns nothing beside the live strand: `shuttle: attach: could not remove the superseded strand <guid> of run <runDir>: <cause>; way forward: run "lyx reed remove <guid>", then re-step the row`.
Update `AttachGated`'s doc comment and the file comment of `attach.go` to state the removal and its bound.

Bound: it removes only strands whose run record declares the exact output-file set of the new run and whose run the probe judged respawn-eligible, so it never touches another producer's strand, a parallel same-role strand with other outputs, or a run the probe would attach to; a probe that refuses (errored or multiple attachable candidates) or attaches removes nothing, and the zero-candidate and reed-state-gate paths have no strand to remove.
Removing strands, never worktrees, keeps `PATTERN-pair-teardown` and `PATTERN-batten-bookend` untouched.

Add a row to `contracts/specs/refusal-spec.md`'s loom section, per `PATTERN-refusal-way-forward` (the error reaches `lyx loom step` and `run` as a producer failure): `producer re-run: superseded strand not removed`, transient, way forward `run "lyx reed remove <guid>", then re-step the row`.

Tests:

- `attach_test.go`: extend `TestAttach_OutcomeDisposition` (or add one table-driven test beside it) with rows over the fake reed: a terminal-outcome candidate on a live tracked strand is removed and Attach answers not found; a dead-pane candidate and an untracked one remove nothing; a candidate with another output-file set is never matched and its strand survives; an attachable candidate is attached to, never removed; a `RemoveStrandErr` makes Attach return the error naming the guid and `lyx reed remove`, with no not-found answer.
- `internal/loomcli/smoke_attachprobe_test.go` (tmux tier, the CLI-level check): beside `TestSmokeBurlerRound_AttachesToALiveRoundInsteadOfRespawning`, a test that leaves a halted round's run record at a terminal outcome with its shell strand still live, re-runs the producer through the same real `Runner`, and asserts the earlier strand is gone from `lyx reed status` before the new strand is added, while a live strand with other outputs survives.

**ImpactSummary:** AttachGated's not-found return now tears down live respawn-eligible strands of the same output set; every shuttle-backed producer row inherits it, and attach or refusal paths are unchanged.
