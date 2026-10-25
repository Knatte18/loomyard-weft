```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 proposal (the `gate_halt`/`gate_halt_at` scope mark), which the fixer carried out, by building a case where it goes wrong.

**[MEDIUM] Stale `gate_halt_at` marks a later entry as a gate-halt resume after a goto or reroute**

- **Changed text:** Card 11, step 7.
  - "An entry is the resume from such a halt when `gate_halt_at` is set and `Status`'s last history entry is a Stuck of this row ... at or after it"
  - "Such an entry that passes the dirty check clears both fields ... no other entry runs it, so ... a reroute entry ... run[s] no verify gate"
- **Problem:** Only a resume entry clears the fields. If the run leaves `Drift-Verify` while the gate halt stands, the fields stay set. A later Stuck of this row from another cause then passes the history test, so the gate runs on commits that no `Drift-Verify` session made. That breaks the "no other entry runs it" guarantee the fix states.
- **Concrete case:**
  1. `Drift-Verify` returns a test-gate Stuck at T1, which writes `gate_halt_at`=T1.
  2. The parent or arbiter moves the run to Code with `goto` instead of resuming. `lyx loom goto` exists and the arbiter has a `goto` decision (`internal/loomcli/arm.go:68`, `internal/loomengine/arbiter.go:159`).
  3. Code commits, and the run comes back into `Drift-Verify`. Its last history entry is another row's, so this entry is not a resume. It runs no gate and does not clear the fields.
  4. The entry stops at step 5 or 6 (no counted capture, compare undeclared), a Stuck of this row at T3 ≥ T1.
  5. On `lyx loom resume`, `gate_halt_at` is set and the last history entry is this row's Stuck at or after it. The entry counts as a gate-halt resume. `AttemptHead` is still the old agent attempt's head, so it differs from HEAD, and the gate runs on Code's commits.
  - The same stale state follows an interrupt between the `UpdateScope` write and the shed persisting the Stuck.
- **Evidence:**
  - Card 11 step 7 clears the fields only inside the resume branch.
  - Only step 9's fresh spawn moves `AttemptHead`, through `ResetRuns`, so steps 5 and 6 leave it stale.
- **Suggested change:** Every entry that reaches step 7 and is not a gate-halt resume also clears both fields. A gate halt that is still standing does not lose its mark this way. An entry after a step 4, 5 or 6 halt that followed the gate halt still passes the history test, because its last history entry is this row's Stuck after `gate_halt_at`, so it counts as a resume and runs the gate.

Tool calls beyond context:
1. Grepped `internal/loomcli`, `internal/loomengine` and `internal/shedengine` for a goto or reroute verb, to check whether a halted run can leave `Drift-Verify` without a resume. The context had no loom CLI verb list.
