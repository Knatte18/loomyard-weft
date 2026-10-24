---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/gitexec/doc.go:22-35
    summary: The Stall bound section was inserted between the pause-hold sentences, so "The hold only delays the call's start and never fails it." now ends the Stall bound section and has lost its referent
    fix: In internal/gitexec/doc.go, move the sentence "The hold only delays the call's start and never fails it." to directly after "Before the deadline starts, a remote subcommand waits while a pause covers the process, ..." and above the "# Stall bound" heading, so the pause-hold paragraph is whole again.
    origin: lens:code-affected-set
---

### [LOW] Stall bound section splits the pause-hold paragraph

**Location:** internal/gitexec/doc.go:22-35
**Issue:** The `# Stall bound` heading and its paragraph sit between "Before the deadline starts, a remote subcommand waits while a pause covers the process, ..." and "The hold only delays the call's start and never fails it."
The second sentence explains the pause hold, but it now closes the stall bound text, right after the line on the operator's `GIT_TRACE_PACKET`.
A reader takes "the hold" to be part of the stall check.
The task's change and the parent's `GIT_OPTIONAL_LOCKS=0` change otherwise compose correctly in this file.

## Interaction verdicts for the focus questions

- `childEnv` and `runCoreWithin`: the environment is composed correctly for remote and local children.
  `childEnv` filters an inherited `GIT_OPTIONAL_LOCKS=`, appends `noOptionalLocksEnv` once, then appends `extra`, which never holds `GIT_OPTIONAL_LOCKS`.
  So there is no duplicate and no dropped entry.
  The capacity hint `len(environ)+1+len(extra)` is exact.
  An inherited `GIT_TRACE_PACKET` stays earlier in the slice than lyx's entry, and `os/exec` keeps the last duplicate, so lyx's entry wins for remote children.
- Tests: the parent's `TestRun` "turns optional locks off" row covers a local child, and the remote environment row covers the remote path.
  They cover different call paths, and the remote row's `GIT_OPTIONAL_LOCKS=0` entry checks presence only, so this is not duplicated coverage.
- Docs: the parent's three `GIT_OPTIONAL_LOCKS` lines in `# Environment` and the task's stall paragraphs agree, and the doc comment on `runCoreWithin` matches the code.
  The one defect is F1.
- Retry path: each attempt calls `childEnv` anew with a new trace path, so nothing from the first attempt reaches the retry.
  An inherited `GIT_OPTIONAL_LOCKS` or `GIT_TRACE_PACKET` cannot leak differently from the first attempt.

## Rejected

- A stall timer firing at the instant git exits non-zero with an empty trace reports a stall and retries once: the window is microseconds, an empty trace means the remote never answered, and the retry is harmless.
- The retry's environment has no direct test: each `runAttempt` rebuilds it from `childEnv`, so the behaviour follows from the code, and the diff adds no new gap.
