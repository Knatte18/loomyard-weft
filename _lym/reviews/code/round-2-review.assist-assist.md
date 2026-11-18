---
verdict: APPROVED
findings:
  - id: F6
    severity: LOW
    class: design
    location: internal/shuttleengine/run.go:904
    summary: The deadline-bound prompt send can still submit an Enter at or after the run deadline through confirmPasted.
    fix: In internal/shuttleengine/send.go confirmPasted, prevent guardedEnter once closeAt has arrived, while retaining successful landing observations and the existing unlanded failure handling. Extend the startup deadline row to assert fake-clock elapsed time, and cover a startup or retype paste begun before the deadline whose box still holds the prompt when confirmation reaches the deadline; assert no Enter at or after it.
    carries: F6
    origin: lens:code-correctness
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: open
    answer: The send now receives the deadline, but confirmPasted can send Enter before checking that the window closed. The new tests do not pin the in-progress cutoff.
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
---

### [LOW] Paste confirmation can submit after the run deadline

**Location:** internal/shuttleengine/run.go:904

**Issue:** The startup send and retype pass their deadline into the verified send.
However, `confirmPasted` sleeps to the confirmation boundary and may call `guardedEnter` before testing `windowClosed`.
A prompt pasted shortly before the deadline can therefore receive another Enter at or after that deadline.
The relevant order is in `internal/shuttleengine/send.go:763`, `:789` and `:799`.
Claude uses this paste path.
This leaves prior F6 partly unresolved.

The tests lens independently found that the startup deadline row checks attempts and typings, not elapsed time.
Removing deadline propagation still leaves one full idle wait, no typing, and no second attempt.
The attach deadline row checks refusal to begin a retype, rather than cutting an in-progress send short.
These are coverage aspects of the same carried finding.

All other focus questions are satisfied by the changed implementation, regression rows, and updated documentation.
The scope and affected-set lenses returned no findings.

The targeted gate could not start because the sandbox denied writing its holder record under the hub's gate directory.
No test pass is claimed from this review attempt.
