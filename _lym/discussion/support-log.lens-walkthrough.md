```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism on one concrete case and checked the end state against the decision's stated purpose.

**Findings**

- **BLOCKING — D12 / D11 ordering.** D12 can never list the archives D11 makes for the same adopt.
  - Evidence: D12 reads origin's archive tags right after the `reconcile` step (step 1).
  - D11's archive runs on the old owner only once its dobby reads the new owner, and the new owner is written at step 4 of D3 and then pushed.
  - Case: B adopts root from A. B's worker reads the tags at step 1, when A has archived nothing yet. A's dobby archives later. B's envelope shows only tags from earlier owner changes.
  - So the purpose "the new owner sees what the old owner had not pushed" is not met by this mechanism. The archives need a later reader, such as `lym beam status` or a re-read when the record ends.

- **MEDIUM — D11 trigger.** The record assumes the old owner's local Bolt receives the new owner, and nothing establishes that it does.
  - Evidence: the dobby's `Owner` dep reads `boardengine.ReadOwners(boardDir)`, the local board (`internal/beamcli/dobbydeps.go` ~104/130).
  - I found no periodic Bolt pull in `internal/dobbyengine`, `internal/beamcli` or `internal/orchengine`.
  - So an idle old owner whose Bolt never pulls never sees the foreign owner, and D11 never fires. This is a fact the design rests on, and it is left unresearched.

- **MEDIUM — D3 / D5: today's "Step 5: the settled runs" has no place in D3.**
  - Today's step 5 calls `resumeSlug` after the owner write (`adopt.go:408`). For each slug adopt did not hold, it lifts the live `OwnerCaller` entry through `tenderengine.Lift`.
  - D5 leaves two kinds of slug unpaused: a run whose pair is live here (`running here`), and a slug holding an entry of another caller.
  - Case: a running-here run with an owner entry. If the lift is kept, adopt lifts the owner pause and the dobby drives the run, which is adopt starting work. If the lift is dropped, D3 does not say so.
  - The record never says whether that lift survives, or in which step.

- **MEDIUM — D2 × D10: an unrelated unfinished adopt blocks every `lym s`.**
  - Case: an interrupted adopt of beam X stands on this machine. The operator runs `lym s` in the prime, which root already owns here.
  - `adopt --here` names `root`, which differs from X, so D2 refuses. `runArgvs` stops, so no `beam start` and no attach.
  - `lym s` stays broken until `--discard`, even though nothing needs adopting. Neither decision names this interaction.

- **MEDIUM — D4 × D10: an `--here` adopt that dies leaves a self-owned subject idle.**
  - Case: `lym s` in a prime this machine owns, and the `--here` worker is killed during `reconcile`.
  - The record reads `interrupted` and names `root`. D4 then keeps the prime's dobby from starting any pass or task, although the machine owns root.
  - D4's bound says "until resumed or discarded", but for `--here` the only case where suppression is needed is the foreign-owner one. Every `lym s` also suppresses the dobby for the length of its reconcile.

- **MEDIUM — D11 step 1 retry: "starting no pass" has no mechanism.**
  - While an archive keeps failing, the dobby stays up on a foreign-owned subject.
  - Today's `planPass` still starts a full pass on a probe tick for an owned-elsewhere run that no pause covers. D11 orders the archive before the owner pause, so no pause covers those runs yet.
  - The record says "starting no pass" but names no suppression in the dobby, unlike D4's.

- **LOW — D2 resume of `beam` / `check`.**
  - The record keeps moved runs, stopped projects and the paused slugs.
  - It does not keep which subjects' owner writes stood. The `beam` and `check` steps start only for "each adopted beam whose owner write stood".
  - A resume at `deploy` or `beam` must re-derive that from Bolt. The record should either list it or say the re-read is the source.

- **LOW — D2 with `--all`.** A re-run of `--all` can compute a different subject set, for example after a beam landed meanwhile. D2 then refuses it as "other subjects", so an interrupted `--all` cannot be resumed by repeating it.

- **LOW — D1 / D2 start race.**
  - The caller cannot tell its own worker's fresh record from the previous `done` record until the worker writes. The record states no bound for this, unlike the check's spawn bound in `internal/adoptcli/doc.go`.

**Per-decision walkthrough**

- D1: caller in an orch pane spawns the worker, then the orch dies mid-deploy. Purpose met: the worker runs in its own session and the record carries the result.
- D2: worker killed after `pairs`, then a same-subject re-run. Met for that case. Broken across subjects with D10, and for `--all` (LOW).
- D3: the order holds, but today's settled-runs lift is missing from the step list (MEDIUM).
- D4: record names root, and the reconcile pulls A as owner. The watcher keeps the orch and the dobby stays idle: met. A dead `--here` record over-suppresses (MEDIUM).
- D5: a run with an `OwnerCaller` entry becomes an `adopt` entry and is `Held`. A green check lifts nothing, `tender resume` lifts it, and the dobby then starts the full pass: met.
- D6: owner written, deploy fails, record stopped at `deploy`, runs paused, no dobby: met.
- D7: `--no-orch` from an orch strand starts the dobby through `BeamUp` and no orch: met.
- D8: the green end only removes the fix task and resets tries: met.
- D9: an `interrupted` record shows its step and the re-run: met.
- D10: `lym s` in a prime A owns runs the adopt, then `beam start` and attach: met. Blocked by an unrelated unfinished record (MEDIUM).
- D11: the old owner's dobby reads the new owner only through its local Bolt (MEDIUM), and its retry path can start passes (MEDIUM).
- D12: not met, because it reads before the archives can exist (BLOCKING).

**Tool calls**

1. Grepped `internal/beamcli/dobbydeps.go` for the owner read and `internal/adoptcli/adopt.go` for `resumeSlug`. My context lacked the source of the dobby's owner read and where today's settled-runs step runs.
2. Grepped `internal/dobbyengine`, `internal/beamcli`, `internal/orchengine` and `internal/fabricengine` for a periodic Bolt pull. My context lacked whether the old owner's local Bolt ever learns the new owner.
