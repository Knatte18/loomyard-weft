# Card 9 — Record the after-speed timing on the pre-retag test set

**Prosa:**
- `docs/benchmarks/test-suite-timing.md`

**Uses:**
- `cmd/testtiming/main.go`

**Intent:**
Re-run card 2's measurement on this tree, after the speed fixes (cards 3–8) and before any retag (D4), with card 2's method: `go build ./...`, three warm runs of each tier, median wall time, load average before and after.
Add the results to card 2's dated section: revision, load, Tier 1 and integration wall time, and the slowest-tests table of the new integration run, beside the before-state numbers.
The integration wall time must come out lower than the before-state (acceptance);
if it does not, stop and find the cause before the retag cards run, since after them the tier's test set changes and the comparison no longer holds.
The PR description later quotes both numbers from this section.
