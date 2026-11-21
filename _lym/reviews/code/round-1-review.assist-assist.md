---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/hardenshed/reap_test.go:115
    summary: Regression coverage does not pin written-root retention or resolution-error fallback.
    fix: Add cases to TestReap with a linked root and a process naming only its written spelling, and an absent root with a process naming that written root. Assert successful completion and the exact ended PID set in each case.
    origin: lens:code-tests
---

### [MEDIUM] Written-root retention and resolution failure lack coverage

**Location:** internal/hardenshed/reap_test.go:115

**Issue:** The new regression supplies only a successfully resolved cwd.
The existing written-root cases use existing roots without links.
They would still pass if matching discarded the written spelling after successful resolution,
or if resolution failure returned an error instead of retaining the written spelling.
These are separate outcomes of the new resolution branch in `rootProcesses`.

The reviewed range is `13cebe3d2ad9925b62c0d1595ce7ec38e6ccc1cc..5ee51579ec4e87e39b7ab70ac2fde89a704a5a12`.
The root match retains the sandbox suffix after resolution, so sharing its parent target does not select sibling processes.
Resolution failure introduces no empty or partially resolved candidate.
The fake orphan has neither an executable nor arguments naming the root; its expected ended PID is independent.
The Linux CLI test uses cwd-only identification and checks exit and reported PID.
The root-cause trailer identifies the path mismatch the implementation changes.
The changed comments describe the resulting match, and every change traces to the requested repair.

Validation was attempted with `timeout 3600s lym gate test --tier 0 ./internal/hardenshed`.
The gate exited before tests ran because its shared holder-record directory under `_bolt/.lym/gate` was read-only.
No fresh passing test result is claimed.
