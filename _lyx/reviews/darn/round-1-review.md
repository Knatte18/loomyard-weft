---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: internal/reedengine/bindings.go:4
    summary: The file header says the re-pin happens when a newer build finds an older build's stamp, but an absent or unparseable stamp also re-pins
    fix: In the file header of internal/reedengine/bindings.go, reword the added line to say every op under the blocking op lock re-pins when the stamp is absent, unparseable or another build's with a commit time not after the running build's, matching doc.go.
---

### [NIT] Header understates when the re-pin fires

**Location:** internal/reedengine/bindings.go:4
**Issue:** The added header line reads "re-pins when a newer build finds an older build's stamp".
`bindingsStale` also re-pins on an absent or unparseable stamp, which is the case that fixes a server booted before this change, and `doc.go` states the full rule.
The header is the shorter of the two descriptions and leaves out that case.

## Checked and found sound

- Range: `product.parent` is `main`; the diff is `internal/reedengine` plus one `test-budget.yaml` row.
- `withOpLock` is the one site every blocking op passes (`Status`, `OpenWindow`, `AddStrand` and the rest), and the re-pin runs under the lock and ahead of `fn`.
  `withTryOpLock` and the name-repair loop stay off it, as the summary and `doc.go` say.
  No nested `withOpLock` exists, so the lock is not re-entered.
- With no server up, `display-message` fails, `repinStaleBindingsLocked` logs at Debug and returns, and the boot pin then stamps.
- `bindingsStale`: absent, garbled and unparseable stamps are stale; the running build's own stamp (trailing newline trimmed) is not; an older or same-commit-time other build's stamp is stale; a later commit time is not; an unstamped running binary never re-pins or reads.
  The newer-wins refinement goes beyond the entry's "different build" and is recorded in the summary and `doc.go`, and deploys move forward in commit time.
  Follow's default is untouched.
- `pinBindingsLocked` sets or unsets the stamp ahead of the `bind-key` calls, and the `set-option` counts in `attach_test.go` and `windowsize_test.go` follow that one added call.
- `TestBindingsBuildStamp` covers the rule table, the stamp the pin writes and the op-lock behavior against the fake tmux without repeating an existing test.
  The `test-budget.yaml` row rises by one for the one new top-level test.
- Every claim in `_lyx/landing/summary.md` matches the diff.
