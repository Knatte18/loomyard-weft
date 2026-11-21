# Red-first gate cannot judge a diff that also changes non-Go files

`red-first-overlay-non-go-files` · Task · bug, gate, testmap · running · Code-Burler-Fix

## Seen

2026-11-19: in `raddle-slug-length-check`, the red-first gate (landed with `bug-red-first-gate`) could not judge the diff because it also changed `contracts/specs/raddle-proposal-spec.md`.
The base overlay cannot put that file back, so the run asked the parent for a "No test seam" waiver although the bug has a Go seam: `TestRaddleBoardContext` and the new `TestCheck` rows fail on the merge base.

## Idea

The overlay restores only the Go files the red-first check needs on the merge base and leaves non-Go files that no test reads at HEAD,
or it puts back any tracked file, so a doc or spec update beside a fix no longer waives the gate.
