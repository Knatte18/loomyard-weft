# Card 15 — after-state

**Prosa:**
- `docs/benchmarks/test-suite-timing.md`

**Intent:**
Decision D10: record the `test-tiers` after-state in `docs/benchmarks/test-suite-timing.md`, beside `test-suite-measure`'s, by the same method: `go build ./...` first, four runs per tier through `cmd/testtiming`, the first discarded, the median of three, with the yaml block of machine, revision, load averages, per-tier top-level test counts and wall times.

It also records:

- the number of test binaries that link tree-sitter under D8's tag set (`go list -test -deps -tags integration,tmux,llm`), before and after the cut;
  this plan's measurement on the before tree counted 44 test binaries (the decision record's 35 was an earlier count), and the after count is taken on this card's tree;
- the link-time measurement behind the seam choice: `go test -c` with a forced relink per package, which on the before tree took about 1.3–1.4 s for a binary linking tree-sitter through `websterengine` (`configreg`, `hubgeom`) against 0.45–0.85 s for comparable binaries that do not (`verifytree`, `fabricengine`), and 3.0 s for `websterengine` itself; the after-state repeats it on the after tree;
- that the `tmux` tier passes on the after-state tree, `TestSmokeDotFillCrossClientControl` included.

The whole suite's top-level test count and the integration tier's wall time, before (`test-suite-measure`'s before-state) and after, go into the PR description as well, which Describe writes from this document.
