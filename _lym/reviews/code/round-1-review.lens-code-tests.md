```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it**

- **The new measurement-entry rule in `darn-fragment-make-change.md` item 3:** no test pins it, and none is needed. It is a stencil's static prose, so it is exempt. Assertions added to the `TestDarnSpec` row "first spawn renders the prior-work token as none" now check this prose (see finding 1).
- **The new measurement-entry rule in `darn-fragment-make-change-gated.md` item 3:** the same holds. It is exempt static prose, and the `TestDarnSpec` row "a cold fixer on the verify gate has the writer and its subagents run no gate" now asserts it.
- **`DarnTable` choosing the gated or ungated fragment from `in.GateFixed`:** the diff leaves this behaviour unchanged. It is already pinned by `ungatedTestStep`, `ungatedSubagentGate` and `gatedNoGateStep`, through `wantInPrompt` and `wantNotInPrompt` in the same two rows (`internal/darnengine/spec_test.go:121-123`, `:149-151`).
- **Header comments in both fragments, and doc comments in `contracts/stencils/stencils.go` and `internal/darnengine/spec.go`:** these are comments with no behaviour, so no test applies.
- **Go behaviour added or changed:** none. The diff changes no Go code path, only comments, a test constant and test rows.

**Findings**

1. **BLOCKING · design · `internal/darnengine/spec_test.go:85-86`, `:121`, `:149`**
   - Summary: the new `measurementEntryRule` assertions pin a stencil's static prose. This breaks `PATTERN-test-speed` and the rubric's Test-economy bullet "a new assertion on the static prose of a stencil".
   - Evidence: `pattern/PATTERN-test-speed.md:284-288` says "No test pins a stencil's static prose", and lists the only things a stencil test may pin: "a marker set, a Fill round-trip or a Go-filled marker's position; a value Go fills, reads or names, a fragment Go chooses, or a rule across a stencil family. Review guards the wording."
   - Why it is static prose: the constant (`spec_test.go:86`) is the opening words of a sentence that appears identically in both fragments (`darn-fragment-make-change.md:23`, `darn-fragment-make-change-gated.md:41`). Go neither fills nor names that sentence.
   - Why it is not "a fragment Go chooses": the rows' existing `ungatedTestStep` and `gatedNoGateStep` assertions (`spec_test.go:80-82`, `:121-123`, `:149-151`) already pin which fragment Go picks. The new string is present in both fragments, so it cannot tell them apart.
   - Fix: remove the `measurementEntryRule` constant and its two `wantInPrompt` entries.

2. **LOW · design · `internal/darnengine/spec_test.go:121`, `:149`**
   - Summary: the new assertions are redundant as coverage of fragment selection, which the same rows already cover through `ungatedTestStep`/`gatedNoGateStep` and the `wantNotInPrompt` lists.
   - Evidence: `spec_test.go:121-123`, `:149-151`.
   - This is the same defect as finding 1, seen from its redundancy side; one fix removes both.

Tool calls: none. My inherited context already held the test file, both fragments and the PATTERN-test-speed rule.
