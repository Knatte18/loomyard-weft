```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: check that the F1–F4 change set agrees with its sibling docs, comments and callers.

**LOW — `pattern/PATTERN-pair-teardown.md:14` is stale after F3.**
- The line lists the `EndForCancel` phase as owned processes, then strands, loop, run-lock wait and session end.
- F3 added the drift capture job termination between the caller's hook and the strand stop. `internal/pairteardown/doc.go` and `cancel.go` now state it.
- The authoritative background file does not mention it. A reader of it would not know cancel ends the capture job.
- Evidence: `PATTERN-pair-teardown.md:14`, against `cancel.go:49-50` and `doc.go:107-108`.

Everything else I checked is consistent:
- **F1:** `RunCompare`'s doc comment, the `CompareRequest.LockRetry` comment, `doc.go:37`, and both production callers (`driftcli/cli.go`, `loomcli/driftwiring.go`) agree. Both set `LockRetry` and use their own named constant, as capture does.
- **F2:** No other file repeats the "partial scope" ruling wording.
- **F3:** The `cancel.go` doc comment, the `doc.go` step renumbering (including the "step 7" cross-reference) and the new `cancel_test.go` rows agree.
- **F4:** The `Head` field is gone from `CaptureRequest`, and no stale reference remains in the code or docs I searched. The side-capture caller at `cli.go:294` never set `Head`, and `RunSideCapture` never read it.

Tool calls made:
1. Grepped Go and markdown files for `CaptureRequest`, `capture job`, `killCapture` and `compare marker`, to find siblings and cross-references my context lacked.
2. Listed the markdown files naming `strand stop` or `EndForCancel`, and the `Head` and capture mentions in `docs/`, to find documents describing the cancel order.
3. Grepped `PATTERN.md` and `pattern/PATTERN-pair-teardown.md` for `EndForCancel`, to read the cancel-phase wording.
