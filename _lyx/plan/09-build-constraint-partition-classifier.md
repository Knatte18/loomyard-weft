# Card 9 — build-constraint-partition-classifier

**Create:**
- `internal/planglyph/buildenv.go`
- `internal/planglyph/buildenv_test.go`
- `plan:internal/planglyph#buildPair` -> `type buildPair struct`
- `plan:internal/planglyph#pinnedBuildPairs` -> `var pinnedBuildPairs []buildPair`
- `plan:internal/planglyph#maxFreeTags` -> `const maxFreeTags = 4`
- `plan:internal/planglyph#constrainedCandidate` -> `type constrainedCandidate struct`
- `plan:internal/planglyph#ambiguity` -> `type ambiguity struct`
- `plan:internal/planglyph#classifyAmbiguity` -> `func classifyAmbiguity(worktreeRoot string, candidates []quarry.Symbol) ambiguity`
- `plan:internal/planglyph#candidateConstraint` -> `func candidateConstraint(name string, source []byte) string`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Add the one classifier that tells a constraint-partitioned member from a genuine ambiguity (decision D6), as a pure addition no policy reads yet.
`buildPair` is one GOOS and GOARCH pair; `pinnedBuildPairs` is the closed list of every pair `go tool dist list` prints today, pinned in source with a comment naming that command as its source.
`constrainedCandidate` is a candidate `quarry.Symbol` with its `Constraint` text; `ambiguity` carries `Partitioned bool` and `Candidates []constrainedCandidate` in the answer's order.
`candidateConstraint` describes one candidate file's build constraint for a finding detail: the file's `//go:build` line verbatim, else its `// +build` lines, else the GOOS or GOARCH filename suffix (such as `_windows.go`), else `no constraint`.
`classifyAmbiguity` reads each candidate's file once (`Symbol.File`, joined under `worktreeRoot`, never derived from a glyph) and reports the answer partitioned when no single build environment selects two candidate files.
The environments are every pinned pair, cgo on and off, and every assignment of the free tags the candidates' constraint lines mention, each evaluated with `go/build.Context.MatchFile` on a zero `Context` carrying only `GOOS`, `GOARCH`, `CgoEnabled`, the tags assigned true in `BuildTags` and an `OpenFile` serving the cached bytes; `Compiler`, `ReleaseTags` and `ToolTags` stay empty, so `gc`, `gccgo`, `go1.N`, `goexperiment.*` and `amd64.vN` are free tags evaluated both ways.
A free tag is a mentioned tag that is no GOOS or GOARCH name of the pinned list, `unix` or `cgo`; `boringcrypto` is assigned under the name `goexperiment.boringcrypto`, which the matcher looks up; `unix` and the GOOS implications come from the GOOS through the matcher.
Mentioned tags are read with `go/build/constraint` from the `//go:build` line, or the `// +build` lines when the file has none.
Fewer than two candidates, a candidate set naming more than `maxFreeTags` free tags, a file that cannot be read, or a constraint line that cannot be parsed yields not partitioned, with every candidate still described.
Document the cost bound in the file doc: one read per candidate file, then in-memory matcher calls, at most the pinned pairs times two cgo values times sixteen tag assignments per candidate.

Tests: `buildenv_test.go` holds one table-driven test over fixture files written to a temp root: a `//go:build linux` file beside a `//go:build !linux` file is partitioned; `_linux.go` beside `_windows.go` is partitioned; `//go:build unix` beside `_windows.go` is partitioned; `//go:build linux` beside `//go:build amd64` is not; `//go:build !windows` beside `//go:build unix` is not; `//go:build !go1.21` and `//go:build !gc`, each beside an unconstrained file, are not; a `// +build linux` file beside a `// +build !linux` file is partitioned; a set naming five free tags is not; an unreadable candidate file is not; a candidate whose `//go:build` line does not parse (`//go:build linux &&`) beside a `//go:build !linux` file is not; one candidate is not; two candidates in one file are not; and each row asserts every candidate's rendered constraint.
The test adds one top-level function to `internal/planglyph`, so raise its row in `test-budget.yaml` by one.

**ImpactSummary:** Pure addition in one package plus one budget row; nothing reads the classifier until card 10.
