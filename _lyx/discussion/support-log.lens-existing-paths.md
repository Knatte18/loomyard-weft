```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the existing code paths that read or write the state each decision changes.

**Findings**

1. **BLOCKING — D4 (passes): every pass re-runs the resume-only side effects.**
   - Evidence, Open-Window: `battenshed/doc.go` says "A run resumed past Open-Window runs that command again when batten.open_window is on and no client is attached to the pair's reed session".
     Each pass is a resume over an existing status file.
     So whenever nobody is attached, every probe tick opens a new terminal window per run.
   - Evidence, origin reconcile: the same doc says "A resume … first runs the origin reconcile of the slug's run scope", and an unreachable origin logs a Warn.
     So every pass of every run does a network reconcile, once per `notice_probe_s`.
   - The record is silent on both.
     It needs a rule that a pass dobby schedules is not a resume for these purposes, or that both run once per batten run (or once per episode) through a marker.

2. **BLOCKING — D4 and the cancel, promote and owner-pause paths: a dobby pass can revive a run being ended.**
   - Evidence: `battencli` cancel and `EndRunForPromote` end the slug's processes through `runproc.End(slug)` and then clear the records and the claim.
     `innerrun_owner.go`'s owner pause ends "the processes of every other run".
   - Under D4, dobby runs a pass whenever a status file is written, on every probe tick, and at once when a pause lifts.
     The pass is the process those paths kill, and the next tick starts another one.
     That can happen while the cancel or promote is between its end and its record removal, or before an owner pause is lifted on purpose.
   - D3 covers ownership only, and the record says nothing about how a cancel, a promote in progress or a cancel marker stops dobby from scheduling.
     It needs a decision: dobby schedules no pass for a slug with a cancel marker, a promote in progress, or a beam mark on its pair.

3. **MEDIUM — D4: `pairteardown` reads "a registered batten is alive" as "the run has a lifecycle owner", which is false between passes.**
   - Evidence: `pairteardown/skip.go`'s `EndBeforeSessionDown` ends a pair's processes when "the status history holds a Finalize Done with no registered batten alive" (`registeredBattenAlive` reads `RoleBatten` records).
     Between the Finalize pass and the Land pass, no batten is alive.
     So a session end outside teardown now ends the processes of a run that is about to land.
   - `claim.go`'s `settleClaim` gives `claimHeldGone`, with a gone-process warning, for any running run that has no live batten.
     That is now the normal state between passes.
   - The fix: these readers count a live dobby that drives the slug as a live batten.
   - Survives: `tenderengine`'s `orphanOwner` does not read a pair whose worktree still exists as an orphan.

4. **MEDIUM — D4 (`--window` retired) against `lyx beam land`.**
   - Evidence: the `beamcli` doc says `land` "starts the landing with `battencli.ResumeInWindow`, whose window opens in the beam's parent worktree's session".
   - D4 retires `--window` and D3 has the parent's dobby drive the land run, but the record never says that `beam land` hands the run to the parent's dobby instead of `ResumeInWindow`.
     The orch role's "Landing halts" and "After landing" text, which mentions windows, is also not covered.

5. **MEDIUM — D5: draft is guarded only in dobby, and existing paths bypass it.**
   - A hand-run `lyx batten run <slug>` is the documented fallback in D4.
     The record does not say that batten's start guard refuses a `draft` task, so a hand run starts a draft.
   - Batten's claim writes a run status through `boardengine.SetOwners`' claim status, which silently overwrites `draft`.
   - The bound "a task is started only by its parent's dobby, only from an empty status" is enforced in dobby only, not at the start guard.
     It needs a start-guard refusal naming `lyx board set-status`.

6. **MEDIUM — D8 and D9: batten's own row notices do not wait for the new batten arbiter.**
   - Evidence: `rownotice.go` posts the Land and Worktree-Teardown failure notices once per failure episode, straight away.
     Only the child's stops have the eligible-stop hold (`stophold.go`), and that hold reads the child's `ArbiterStop`, not batten's own run.
   - Once D8 makes batten's stops arbiter-eligible, the orch is told about a Land failure before the arbiter has had its turn.
     That contradicts D9's "kept, since each needs judgement", which is meant to apply after the chain has run.
   - It needs a hold on batten's own row notices, matching the child's.

7. **LOW — D1: `RoleDobby` is registered under the beam's slug, and that slug is also the beam land run's owner.**
   - The land run's own `runproc.End(beamSlug)` calls (its teardown, an `EndBeforeSessionDown`, an owner pause) end the beam's dobby mid-landing.
     That is harmless at teardown, but an owner pause of a beam that is landing ends its dobby.
   - Survives in substance; the record should say it.

8. **LOW — D4: the watched marker.**
   - `loomcli/start.go` and `drivewiring.go` read the marker through `proc.IsAlive`, a pid liveness check only, so dobby's pid reads right.
   - Survives, but `wire.go` writes the marker with `os.Getpid()`, which is the pass's pid.
     The record states the dobby pid, so the writer's call site must change; this is noted only as confirmation that the change is real.

**Paths traced, per decision**
- D1: `runproc.End` from pair teardown survives; the owner pause of a landing beam is LOW (7); `tender` `orphanOwner` survives.
- D4:
  - Open-Window re-run on resume breaks (1).
  - Origin reconcile on resume breaks (1).
  - Cancel, promote and owner-pause end, then a rescheduled pass, breaks (2).
  - `pairteardown` `EndBeforeSessionDown` and `settleClaim` break (3).
  - `beam land`'s `ResumeInWindow` breaks (4).
  - Watched-marker readers survive (8).
  - `ClearRunMarkers` on a fresh seed survives.
- D5:
  - Batten start guard and `SetOwners` claim break (5).
  - README Ready and `IsRunStatus` survive (draft is not a run status).
- D8:
  - Batten row notices break (6).
  - The `loom.yaml` override semantics against the hub base and `hubreconcile` survive (0 off and 0 for 30 minutes keep their meaning).
- D12: batten's in-pass pause handling survives; the Land row under a pause still halts blocked, as today.

**Tool calls**
- grep for `RoleBatten`: to find which paths read batten liveness; my context lacked the readers.
- `pairteardown/teardown.go`, batten-alive and driver-lock helpers, plus a grep for watched-marker readers: my context lacked how teardown reads batten and who reads the marker.
- `pairteardown/teardown.go` around `battenAlive` and `sheddriver/drive.go`'s `watched`: to see which callers use it.
- `pairteardown/claim.go` and `skip.go`: to see what batten liveness decides in the claim settlement and `EndBeforeSessionDown`.
