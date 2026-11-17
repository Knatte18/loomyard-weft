```yaml
lens: code-scope
outcome: reported
```

The code-scope lens: tracing each change in the diff to a goal point or added entry in the decision record.

**The decisions** (goal points of the `## Goal` board body, plus the record's `Added after Discussion` entries)

- **G1:** hibernate's pause-and-confirm becomes one exported function that the carrier and the stop both call, and it owns only an entry it added.
  - Built by C1 and C2.
- **G2:** `lym beam stop --all` re-starts itself detached with a logged spawn and runs D9's sequence. The steps are pause and confirm, the pairs through reed down's path, BeamDown for the prime and each beam, the beams' reed sessions, Reap, Lift of its own entry, and the prime's reed session last.
  - Built by C5, with C2 and C4 as its seams.
- **G3:** `orchcli.BeamDown` ends the orch's registered Claude process through runproc, as its last step, for every caller.
  - Built by C3.
  - The pawl and cancel callers reach it unchanged through `BeamDown` (`internal/loomcli/pawlwiring.go:166`, `internal/battencli/cancelcmd.go:384`).
- **G4:** a stop entry with confirm text in reed's prime button group, in both templates.
  - Built by C6.
- **G5:** no tracked record, no per-run intent, and a standing Bolt hold left as it is.
  - Built by C5: the stop writes no record and no intent, `Lift` takes only the owned ref, and nothing touches the hold.
- **G6 (Exports line):** the part touches no `internal/dobbyengine`, `internal/machinecli` or `internal/termwindow` code.
  - Satisfied: the diff stat shows none of those packages.
- **Added (parent): no per-run pause or cancel intent.**
  - Built by C5, which writes no intent.
- **Added (parent): only the printed `parent_beam` names the prime root (D8).**
  - Not this part's goal; no change expected, none made.
- **Added (arbiter): round-2 Discussion review accepted.**
  - Reads D9's bound as pushed-or-owed. The diff adds nothing against that (C5's doc states the bound).
- **Added (arbiter): split review round 2 accepted (fixtries, criteria moves).**
  - Not this part's; no change expected, none made.

**The changes**

- **C1:** `internal/hibernateengine` (`watch.go` PauseAndConfirm, the Confirmation/HubPause/PauseConfirmRequest types, carry's refactor, `doc.go`, `pauseconfirm_test.go`). Traces to G1.
- **C2:** `internal/hibernatecli` (`PauseHub` in `deps.go`, `doc.go`). Traces to G1 and G2 step 1.
  - Outside the record's Scope package list, but forced: the engine Deps the stop needs are built only by `hibernatecli`'s unexported `hubDeps` (`internal/hibernatecli/deps.go`).
- **C3:** `internal/orchcli` (`stop.go` endOrch step and the `OrchProcessEnded` field, `doc.go`, `beamdown_test.go`). Traces to G3.
- **C4:** `internal/reedcli` (`wireLocation`, `down`, `DownWorktree`, `DownReport`, `doc.go`). Traces to G2 steps 2, 4 and 7, which go "through reed down's path".
- **C5:** `internal/beamcli` (`hubstop.go`, `hubstop_test.go`, the `cli.go` `--all` and hidden `--detached` flags, `doc.go`). Traces to G2 and G5.
- **C6:** `internal/reedengine` (both templates and `config_test.go`). Traces to G4.
  - The confirm text is identical in both templates (`template_posix.yaml:77-79`, `template_windows.yaml:77-79`).
- **C7:** `docs/overview.md`, the beam module line. Forced by C5 as a stale doc.
- **C8:** commit `ee537d9a`, a comment line-break fix. It touches only files that C1, C3, C4 and C5 already change. Forced by those changes; no new scope.

**Findings**

None. Every change traces to a goal point or is forced by one, and every goal point of this part has a change that builds it.

- "Accepts a pair a hub pause covers" (G2 step 2) needed no code change: reed down's existing path already admits a pair under a pause (`internal/reedcli/up.go:189-193`). Its absence from the diff is no gap.
- The record's D9 text names `runproc.EndSubject`; the diff uses `runproc.EndRole` (`internal/orchcli/cli.go:320-322`). The goal point says only "through runproc", so this is no scope finding against the measuring stick.
  - Note for the handler, outside my lens: `_lym/landing/summary.md` declares this departure from D9's wording.

Tool calls:
1. `git show --stat ee537d9a` and `66026998`: to see which files the comment-fix commit touched, because the manifest held only the combined diff.
