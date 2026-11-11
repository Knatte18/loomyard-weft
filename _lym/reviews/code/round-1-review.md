---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/codexengine/sandbox.go:56
    summary: >-
      Under auto, any probe error drops the sandbox, including a 30 s timeout, a missing binary or a killed probe,
      none of which show that the sandbox cannot start.
    fix: >-
      In runSandboxProbe (sandbox.go:25-40), return a distinguishable error type for a probe that could not run or
      timed out (ctx.Err() non-nil, or the error is not an *exec.ExitError, such as exec.ErrNotFound),
      and keep the plain wrapped error only for an *exec.ExitError, which is the sandbox's own refusal.
      In launchSandboxMode (sandbox.go:56-60), degrade auto to danger-full-access only for the exit-error kind;
      for the other kind return the error from Prepare and OpenLaunch, naming the cause, under auto and workspace-write alike.
      Add a TestRunSandboxProbe row with a nonexistent bin and a TestPrepare_SandboxMode row for the non-refusal kind,
      each asserting the launch is refused and the sandbox is not dropped.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/codexengine/open_test.go:54
    summary: >-
      OpenLaunch's explicit workspace-write refusal (codexengine.go:190) has no test row,
      and no OpenLaunch row records the probe's arguments, so removing that error return fails no test.
    fix: >-
      Add a TestOpenLaunch row whose setup writes codex.yaml with sandbox_mode workspace-write,
      with probeErr set to errors.New("bwrap: setting up uid map: Permission denied"),
      and wantErr holding that cause and "way forward".
      Let the row's probe stub record its bin, home and dir arguments and assert them, as TestPrepare_SandboxMode does.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/sandbox.go:19
    summary: >-
      The probe sets only sandbox_mode, so it runs neither the writable roots nor the network setting of the launch,
      and with an empty dirs.Anchor it runs in the process cwd; the doc claims no such limit.
    fix: >-
      Add one sentence to the runSandboxProbe doc comment (sandbox.go:19-24) saying the probe checks only that
      the workspace-write sandbox itself starts, not the launch's writable roots or network setting,
      and that an empty dir runs it in the process's working directory.
    origin: lens:code-correctness
---

### [MEDIUM] Transient probe failure degrades auto to danger-full-access

**Location:** internal/shuttleengine/codexengine/sandbox.go:56
**Issue:** `launchSandboxMode` treats every error from the probe as "the sandbox cannot start".
A probe killed by its 30 s timeout under load, a binary that is missing or not executable, and a signalled probe all take the same branch as bwrap's real refusal.
Under `auto`, the template default, a transient cause therefore removes the write bound for that seat with only a log warning.
The decision record asks for the mode to be settled by what makes the sandbox fail on a machine like OSL-1046, and a timeout does not show that.
The code-tests lens's note on the untested empty-output branch of `runSandboxProbe` is folded into this finding's fix.

### [MEDIUM] OpenLaunch's explicit workspace-write refusal is untested

**Location:** internal/shuttleengine/codexengine/open_test.go:54
**Issue:** `OpenLaunch` has its own `launchSandboxMode` call and error return (codexengine.go:190).
The only probe-failure row in `TestOpenLaunch` runs under `auto`.
The refusal that Goal item 2 asks for is pinned through `Prepare` alone, so an opened Codex session could lose it unnoticed.

### [LOW] The probe's limits are undocumented

**Location:** internal/shuttleengine/codexengine/sandbox.go:19
**Issue:** The probe runs `codex sandbox -c sandbox_mode="workspace-write" -- true`.
I ran that shape with Codex 0.164.0 under an existing `CODEX_HOME`: the subcommand and the flag exist, and it exits 0 here.
It leaves out the writable roots and the network setting the launch line carries, so it checks the sandbox start and not the launch's full policy.
With an engine from `New()` the dir is empty and the probe runs in the process cwd.
This is probably harmless, since bwrap's namespace setup does not depend on either, but the doc comment does not say so.

## Rejected

- The auto fallback's warning reaches only lym's log (lens code-correctness F3): Goal item 2 asks for a report of a seat that cannot start, and an auto seat does start.
  The change description states this limit under "Not covered", so the claim and the diff agree.
- The untested 30 s timeout (lens code-tests F-tests-2): the timeout is a const and not injectable, and the missing-binary row is part of F1's fix.
- Declared PATTERN changes, commit trailers and stencils: the directive declares none, the commits carry no `Pattern-Departure:` trailer, and the diff changes no stencil.
- The change description's claims on Neve (probe about 35 ms, exit 0): not contradicted by the diff or by my run; its OSL-1046 cause is stated as unconfirmed, which the diff can neither bear out nor refute.
