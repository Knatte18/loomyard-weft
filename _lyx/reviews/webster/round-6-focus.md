---
round: 6
focus:
  - "In `internal/loomcli/circling.go` and `internal/loomcli/decision.go`, enumerate every `refuse(` call and every refusal built from a forwarded helper error, and settle for each whether its message names a way forward, whether `contracts/specs/refusal-spec.md` has a matching row in the loom section, and whether a test reaches it and asserts `way forward:`."
  - "In `internal/shedadapters`, enumerate every Stuck `Reason` and `degrade` message added by the diff that forwards an error from a shared helper, and settle whether each names a way forward, has a shed-section row and is reached by a test."
  - "Run `gofmt -l` over every Go file the diff touches, and settle whether any is listed."
---

The three entries restate the sweep method, applied per refusal site rather than per refusal wrapper, so sites behind a shared wrapper are not missed.
