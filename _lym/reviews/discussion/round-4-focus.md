---
round: 4
focus:
  - "Check that decision-record.md and support-log.md agree with every `### Added after Discussion` entry: D2 must carry no staleness rule that makes testmap count a map stale for dependency drift, and Open risks must not ask the parent a question an Added entry already answers."
  - "Check D3's union of selected tests: can the second member name a test of the tier declared in a test file that build constraints exclude on the gate's host, so the selected `go test -run` invocation runs zero tests in a package that holds other tier tests, and the gate's empty-tier check (`emptyTierFailure` in internal/gatecli/tierrun.go) fails? Settle whether D3 and D6 bound this."
  - "Check D2's repeated dirty check: does the record state that both checks are fresh reads of the worktree status around a package's build and tests, and does it bound the case of a linked file edited after the first read and reverted before the second?"
  - "Check D5's `inert_paths` exemption for a recorded non-Go file of a linked package: can it narrow importers to none selected when the file is named by a `go:embed` pattern in that package, and is that case bounded?"
  - "Check D5's list of changes exempt from a miss: can an added package-level declaration whose name is a predeclared identifier (`max`, `min`, `len`, `new`, `error`) rebind unchanged code of its own package?"
---

The focus lists the round-3 design questions the record answers by text that no fresh reviewer has seen, plus the consistency check against the parent's recorded decisions.
