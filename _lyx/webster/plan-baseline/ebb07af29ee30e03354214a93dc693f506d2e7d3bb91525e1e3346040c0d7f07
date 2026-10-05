# Card 2 — Record the before-state timing

**Prosa:**
- `docs/benchmarks/test-suite-timing.md`

**Uses:**
- `cmd/testtiming/main.go`

**Intent:**
Record the suite's before-state in `docs/benchmarks/test-suite-timing.md` as a new dated section (D3), measured on this tree before any speed fix (cards 3–8) and before any retag (D4).
Run `go build ./...` first, then three warm runs each of `go run ./cmd/testtiming` and `go run ./cmd/testtiming -full -top 20`, and take the median wall time per tier, the method the file already states.
The section names the machine (CPU, OS, kernel), the revision (`git rev-parse --short HEAD`), the load average from `uptime` taken before and after the runs, the total top-level test count per tier, the Tier 1 and integration wall times, and a slowest-tests table of the integration run's top 20 (test, package, elapsed).
Card 9 adds the after-speed numbers to this same section, so it gets a heading that card can extend.
The section is a measurement report and is exempt from the perishable-specific rule (D3);
every link in it resolves (`PATTERN-markdown-link-integrity`), and its prose uses semantic line breaks.
Keep the load comparable to card 9's run: no other loom run or test suite running on the machine, which the section states.
