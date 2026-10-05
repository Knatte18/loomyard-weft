---
round: 5
focus:
  - "Enumerate every call to `refuse(` in `internal/loomcli` and every `Stuck` Reason built in `internal/shedadapters`, and for each new or changed refusal in the diff settle whether its message names a way forward, whether the loom or shed section of `contracts/specs/refusal-spec.md` has its row, and whether a test reaches it and asserts the clause."
  - "Check every refusal in the diff that forwards an error from a shared helper (for example `readEscalation` or the decision append path) rather than formatting its own message: does a malformed-input or transient-I/O error reach the user without a way forward?"
  - "For each refusal-spec row in the diff whose Way forward cell is meant to quote a message, compare the cell to the message string in code and to the substring the test asserts, and settle whether all three agree."
---

The focus asks for a sweep by method over all refusal sites, not by instance.
