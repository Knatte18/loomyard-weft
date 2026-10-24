Task: check the fix's change set for consistency with its surroundings (fix-consistency lens).

**Findings**

1. **MEDIUM** — `internal/gatecli/doc.go:253`, `internal/gatecli/run.go:35`, `docs/overview.md:461`, `internal/gatecli/cli.go:68`.
   - **Summary:** The new `haltExitCode` rule leaves the documented exit-code lists stale.
   - **Evidence:** After the fix, a busy hub (`SlotBusyExit`) following an earlier failed run exits with that failure's code, not 75. A halt that reports 0, such as a cancelled context with no signal, now exits 1.
   - **Stale text:**
     - `doc.go:253` lists "SlotBusyExit (75) for a busy hub" unconditionally.
     - The `runTest` comment at `run.go:35` does the same.
     - `docs/overview.md:461` says the busy hub "exits 75, which is no failure".
     - `cli.go:68` says "The exit code is the first failing run's own, or 0", with no halt rule.
   - **Missing:** The CLAUDE.md "Docs land in the same commit" rule requires the module doc to carry this change. The fix commits touched none of these files.
   - **Suggested wording:** "75 for a busy hub when no earlier run failed, else the first failing run's code; 1 for a halt that reports 0."

2. **LOW** — `internal/gatecli/doc.go` (the gatecli doc describing the closing line).
   - **Summary:** The closing line now also prints on a halt, and the "every later run still ran" wording is gone, but the docs describing the closing line were not updated.
   - **Evidence:** `overview.md:461` says "a closing line names every failed run". That is still true, but nothing documents that a halt (refusal, busy hub) prints it for the runs that failed before the halt. I did not read `doc.go` around the closing-line paragraph, so check for an "every later run" phrase there.

**Calls made**
1. `grep` over `cli.go`, `doc.go` and `overview.md` for exit-code wording, plus `sed -n 33,37p run.go`: my context lacked those docs and the `runTest` comment in their current form.
