No findings.

- F1: `surfaceBeamUpWarnings` has one caller (`pawlwiring.go:110`) and one test (`pawlwiring_test.go:66`). Both use the new `notify` signature, so no stale caller remains.
- F4: `docs/overview.md` lines 421, 539, 678 and 812 are updated.
  - `internal/orchengine/doc.go:14` calls the runs strand "the session's overview window", which is still true. It does not mention the loom-status replace or the warning.
  - Line 677 of `docs/overview.md` shifted to 678 because the fix added a line below it.

Calls:
1. grep for `surfaceBeamUpWarnings` across the Go files: the review named only the definition, so I checked for other callers.
2. grep for overview-window wording across the docs: I checked for other places that describe the old overview-slot behaviour.
