---
round: 1
focus:
  - "Goal coverage: the board entry asks that the tier1 `go test` run get a raised `-timeout` from a config key. The diff adds the `{test_timeout}` placeholder and `test_timeout_min` mechanism in `internal/gateslot/tiers.go`. Settle whether any tier1 command in the repository's own gate config now holds `-timeout {test_timeout}`, or whether the placeholder exists but no `go test` run passes it, so the panic would persist."
  - "The entry says to confirm from the code whether the gate sets `-timeout` at all. Check that the diff's wording and the change description agree with how the gate builds its `go test` command line, including the `tiers.go` renderer in `internal/gatecli`."
  - "`decodeTier` in `internal/gateslot/tiers.go`: settle whether the validation order, the refusal for a command without the placeholder, the below-1 refusal and the default of 20 behave as the commit message claims, and whether `TestTimeoutMin` stays 0 for a command without the placeholder."
  - "`renderTierCommand` in `internal/gatecli/tiers.go`: settle whether `{test_timeout}` renders as `<n>m` on every render path, and whether any other code that builds or re-renders a tier command bypasses the placeholder."
  - "Green fingerprint in `internal/gatecli/green.go`: the commit says `test_timeout_min` is ignored like `timeout_min`. Settle whether `tierFingerprint` hashes the command text with the placeholder unrendered, and whether a changed timeout value can wrongly carry or invalidate a pass."
  - "Test coverage: settle whether each new behavior (default, explicit value, refusal without placeholder, refusal below 1, rendering, fingerprint exclusion) has a test in `internal/gateslot/config_test.go`, `internal/gatecli/green_test.go` or `internal/gatecli/tiers_unix_test.go`, and whether any new test duplicates an existing one."
  - "Repository rules: settle whether the docs the repository's constraints document names for a gate-config change landed in the diff (`doc.go` files, `template.yaml`, any spec under the specs directory), and whether new doc comments follow the repository's comment conventions."
---

Seed focus for round 1. The record holds only the board entry under `## Goal`, so the entry and its `Update` section are the measuring stick.
