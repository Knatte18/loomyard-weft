---
round: 1
focus:
  - "internal/gatecli/child.go `killLeftovers`: it runs after `handle.Wait()` has reaped the shell, on every exit including a clean one. Settle what `proc.KillTree` does with the record then: can it signal a process group whose leader pid was freed and reused, or kill a group that is not the shell's? Read `proc.KillTree` and the `Tied.Record` contract."
  - "internal/gatecli/child.go ErrWaitDelay handling: settle whether `errors.Is(waitErr, exec.ErrWaitDelay) && cmd.ProcessState != nil` returns nil only for a zero exit, and what the callers (`tierrun.go`, `build.go`, `rungate.go`) record for exit code and outcome on that path versus a timeout or cancelled context."
  - "internal/gatecli/child.go ordering: settle whether killing the group only after the wait delay, with the pipe read already cut, still leaves a path where a leftover holds a gate slot or the pipe, and whether the kill also runs on the timeout, cancel and failed-start paths."
  - "Expected section of the decision record, three bullets: a tier or build child ends within a bounded delay with `WaitDelay` set on both commands; leftovers in the process group are killed; the tier records the shell's exit code and fails. Settle which of the three the diff does not carry for the build step in `build.go`, which has no test beside the tier's."
  - "internal/gatecli/test_integration_test.go new orphan scenario: settle whether it asserts the pipe-hang case it names, that is, whether the pre-change code would hang or fail it, and whether it covers the build path or a zero-exit shell with an orphan."
  - "internal/gatecli/doc.go new paragraph and `newCommand` doc: settle whether every claim (10 seconds, process-group kill, Windows job close, the stated bound on a leftover in its own session) is borne out by the code, and whether the spec docs the repository's constraints name for a change to gate behavior land in the same diff."
---

Seed focus from the decision record's `Goal` (the record carries no Scope or Acceptance criteria) and the single commit in the range.
