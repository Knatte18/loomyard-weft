---
format: 5
approved: true
language: go
---

# Plan: the dobby watches every run it drives and mails its parent

The dobby of a run's parent worktree reads each run it drives on every scheduling tick and mails the nearest live orch a `[run watch]` line for every stop, or every change under `beam.run_watch: rows`.
On probe ticks it also notices a gate slot a driven run holds past a bound derived from its tier's `timeout_min`, naming the holder's process tree and any marked process outside it.
The cards add the keys first, then the run watch's engine and wiring, then the holder record's marker, then the gate-hold notice's engine and wiring.

## Card Index

1 — run-watch-setting — add beam.run_watch with its per-task lookup and the gate_hold_factor and gate_hold_min hub keys
2 — run-view-history — carry batten's history length, last repeats and reason in the dobby's RunView, and the child's last repeats in its reading
3 — run-watch-engine — watch each driven run's reading per tick, remember it, and owe and deliver its watch lines
4 — run-watch-wiring — wire the watch's level lookup and untagged post in the hub's dobby and name the lines to the orch
5 — gate-holder-marker — record a tier run's environment marker in its gate slot holder record
6 — gate-hold-engine — notice once per hold a gate slot a driven run holds past its bound
7 — gate-hold-wiring — wire the gate holders, process reads and tagged post in the hub's dobby and name the notice to the orch

## Shared Decisions

- The episode stamp follows the decision record's Added after Discussion entry "Episode stamp counts folded repeats": a layer's history length plus its last history entry's `Repeats`.
- The hub's orch role stencil, `orch-template-role.md` under the hub's stencils directory, also lists the notice prefixes.
  It lives outside the repository, so no card can change it under `card-fabric-reference`; `contracts/orchrole/orch-role.md` is the copy the cards change.
