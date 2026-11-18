```yaml
lens: fix-claims
outcome: reported
```

Task: check, against the code at HEAD, every claim the fix added or rewrote.

- LOW — D7 relaunch step 2: "The engine's `Prepare` builds a fresh resume launch …, so … reed's stored command and `ResumeCmd` all name the new account."
  Summary: `Prepare` does not make reed store anything. It returns the launch (`internal/shuttleengine/run.go:650`), and reed stores the command when the strand starts, which is step 4.
  Evidence: in `OpenLaunch`, `command.go:487-492` returns `Cmd`/`ResumeCmd` in a `shuttleengine.Launch` value and writes nothing to reed.
  The claim only holds once step 4 has run. The fix should say that reed's stored command names the new account from step 4 on.
- LOW — D7 orch paragraph: "`resumeOrch`, which ends the orch's own live holder before it resumes".
  Summary: `resumeOrch` first tries to resume. It ends the holder only after that resume is refused for a live holder, then resumes once more. It does not end the holder up front.
  Evidence: `internal/orchcli/start.go:808-817`.
  The D7 orch acceptance criterion ("ends the still-live limited orch process, then restarts") is met by that refuse-then-end path. Only the wording "before it resumes" is loose.

The other claims hold:
- Shuttle has no in-run relaunch today, and Attach never relaunches (`attach.go:413`).
- The stop verb exists in `internal/shuttleengine/stop.go` (`Run.Stop`, `Runner.StopStrand`).
- The resume check runs before `Prepare` (`run.go:982` vs `run.go:650`).
- `retryAPIError` counts and re-sends a usage-limited turn end (`apierrorretry.go:46,63-65`).
- `OpenLaunch` writes settings only when there is a creel flag directory (`command.go:474-481`), and D5 now states the change.
- The orch start launches fresh on a missing transcript (`start.go:781`).

Out of scope: `attach.go:13,53,204` removes live strands of respawn-eligible candidates.

Tool calls:
1. `grep`/`ls` on `internal/shuttleengine/stop.go`, `internal/shuttleengine/run.go` and `internal/shuttleengine/*.go`. My context did not show the stop verb's existence or the order of the resume check against `Prepare`, which the new D7 step list relies on.
