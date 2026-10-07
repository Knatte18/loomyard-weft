# Card 5 — treesitter-link-guard

**Create:**
- `cmd/lyx/treesitterlink_integration_test.go`
- `cmd/lyx/treesitterlink_test.go`

**Edit:**
- `pattern/PATTERN-test-speed.md`

**Intent:**
Decision D8's guard, so the cut of cards 3 and 4 cannot regrow unseen.
An `integration`-tier test in `cmd/lyx`, since it runs `go list`, carrying `//lyx:guard` on the line directly above its `func Test…` line, runs `go list -test -deps -tags integration,tmux,llm` over the module, through the logged-spawn route `PATTERN-spawn-observability` requires.

It fails, naming the package and one import chain from the test binary to `github.com/tree-sitter/go-tree-sitter`, when a test binary of a package outside the allowed list links tree-sitter.
It also fails when an allowed package's test binary no longer links it, so the list cannot go stale.
The tag set is D2's, so a binary that links tree-sitter only through a `tmux` or `llm` test file counts.

The allowed list is a `scankit.Entry` slice in the test file, each entry carrying its reason.
After cards 3 and 4 it holds the packages that call the code index for real: `planglyph`, `quarrycli`, `webstercli`, `loomcli`, the CLI packages above them (`cmd/lyx`, `shedcli`), and `websterengine` and `loomshed`, whose tests drive the real index through the seam.
The implementer pins the list from the guard's own first run and gives each entry a reason; any further package it reports is either cut at its seam or allowed with the remaining link stated, per the decision record's open risk.

The chain search and the allowed-list comparison are pure functions over the `go list` output, declared in `treesitterlink_test.go` (untagged, so the `integration` file can call them) together with a table test over a hand-built dependency map: a package outside the list fails with its chain, a stale allowed entry fails, and a clean map passes.

`pattern/PATTERN-test-speed.md`'s background states the tree-sitter allowed list and where it lives.

**ImpactSummary:** Adds one guard and its background paragraph; no production code changes.
