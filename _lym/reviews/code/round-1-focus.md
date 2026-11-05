---
round: 1
focus:
  - "internal/fabricengine/clone.go CloneHub, two-argument form: between the offline checkResetTarget and the shared resetHub call, does any step before the probe's last check (warp binding, shortname, old-order guard) still remove or mutate the hub, or return early past the reset so --reset is silently skipped?"
  - "internal/fabricengine/clone.go CloneHub, one-argument form: the hub name now comes from the probe, so confirm the offline non-hub refusal in resetHub still runs before any removal and that the hub-exists check for the non-reset path is unchanged."
  - "internal/fabricengine/clone.go: resetHub now re-stats the target through checkResetTarget after the probe in the two-argument form; check the window between the early check and the removal for a behavior change (a hub that appears, vanishes or stops looking like a hub) and the error paths that follow."
  - "Failure-path coverage against the decision record's Goal (a refused --shortname or warp binding must leave the existing hub untouched): does shortnamebinding_clone_integration_test.go assert the hub survives for each refusal kind in both forms, or only for one, and does it duplicate an existing test?"
  - "Commit messages and internal/fabricengine/doc.go: does the new doc.go sentence and the CloneHub doc comment match what the code does, including the claim that the offline non-hub refusal still precedes the network in both forms?"
---

The change is small (clone.go, one doc.go line, one integration test).
The risk sits in ordering: where the reset moved relative to the probe, and what the two forms now do on each refusal.
