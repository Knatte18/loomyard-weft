# Card 11 — Add the llmkit test kit

**Create:**
- `internal/testkit/llmkit#Claude`
- `internal/testkit/llmkit/llmkit_test.go`

**Edit:**
- `internal/testkit/enforcement_test.go`
- `internal/testkit/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-testkit.md`

**Intent:**
A new kit `internal/testkit/llmkit` holds the one helper that locates the LLM binary for a test and skips the test when it is absent (D7).
`Claude` returns the value of the environment variable `overrideEnv` when that is set and non-empty, else `exec.LookPath("claude")`'s answer, and calls `t.Skip` naming both the variable and `claude` when neither yields a path.
The parameter keeps the per-package override variables the current copies honour (`LYX_REED_CLAUDE`, `LYX_SHUTTLE_CLAUDE`); card 13 moves those copies onto the kit.
The package doc comment, in `llmkit.go`, states the kit's purpose, that only `llm`-tagged test files may import it, and its bounded `os/exec` exemption.

`llmkit` imports `os/exec` for `LookPath` only: the third, bounded exemption from `PATTERN-testkit`'s `os/exec` ban, after `lyxbin` and `tmuxkit`, recorded in `pattern/PATTERN-testkit.md` and, where the overview line names the exemptions, in `PATTERN.md`'s `PATTERN-testkit` entry.
`internal/testkit/enforcement_test.go` admits `os/exec` for `llmkit` alongside the two existing kits, and bars every other kit from importing `llmkit`, as it does for `lyxbin` and `tmuxkit`.
The import check cannot see which `os/exec` identifiers a kit uses; card 15's guard closes that gap for `llmkit`.
`internal/testkit/doc.go`'s kit list gains `llmkit`.

`llmkit_test.go` covers `Claude` without spawning anything, as one table-driven test: the override wins over `PATH`; with no override, a fake executable `claude` in a temp directory set as `PATH` is found; with neither, the test is skipped (observed through a subtest's `Skipped()`).
It sets `PATH` and the override variable through `t.Setenv`, so it does not call `t.Parallel`, and a comment atop the test names that process-global state.
Bound: the kit only locates a binary; it starts nothing.

**ImpactSummary:** a new kit plus its admission in the testkit import check; no production package changes and no existing test changes behavior.
