# Card 15 — Guard LLM spawns into the `llm` tier

**Create:**
- `cmd/lyx/llmtier_test.go`

**Edit:**
- `cmd/lyx/tierpurity_test.go`
- `PATTERN.md`
- `pattern/PATTERN-test-speed.md`

**Uses:**
- `plan:internal/testkit/llmkit#Claude`

**Intent:**
A guard test in `cmd/lyx`, beside `tierpurity_test.go`, keeps LLM spawns in the `llm` tier (D7).
Walking every `_test.go` file in the module through `scankit`, it fails when:

- a test file whose build constraint can be true without `llm` imports `internal/testkit/llmkit`; the constraint is evaluated with `llm` unset and every other tag and platform term set, so `//go:build llm` and `//go:build llm && linux` pass while `//go:build tmux || llm`, `//go:build integration` and an untagged file fail;
- a test file outside `internal/testkit/llmkit` calls `exec.LookPath` with an LLM binary name (`llmBinaryNames`, today `"claude"`) as a string literal or a constant resolvable in the same file;
- a file under `internal/testkit/llmkit`, production or test, uses an identifier of its `os/exec` import other than `LookPath`, which `internal/testkit/enforcement_test.go`'s import check cannot see.

Files under `internal/testkit/llmkit` are exempt from the first two rules, and the guard file itself is exempt as scan data through an allowlist entry with its reason.
The checks parse with `go/parser` and `go/build/constraint`, reusing `tmuxisolation_test.go`'s constraint helpers rather than copying them.
A fixture-tree test in the same file, shaped like `TestTmuxIsolation_FixtureTrees` (temp files scanned through `scankit.Options{Base: ...}`), shows each rule failing on a violating fixture and passing on its clean twin: an untagged and a `tmux || llm` file importing `llmkit`, an `llm` file importing it, an `integration` file calling `exec.LookPath("claude")`, a kit file calling `exec.Command`, and a kit file calling only `exec.LookPath`.
On the real tree after cards 11–14 it passes.
`tierpurity_test.go`'s allowlist gains the new guard file only if its fixtures trip that guard's banned tokens.

`PATTERN.md`'s `PATTERN-test-speed` line adds that only `llm` files reach an LLM, through `llmkit`, and `pattern/PATTERN-test-speed.md` names this guard and its bound.
Bound: the guard sees the static shape only.
A `tmux`-tier test that drives `lyx` into spawning an LLM without calling `llmkit` is caught by review, which the background states.

**ImpactSummary:** test-only: one new guard in `cmd/lyx` plus PATTERN text; at most one allowlist row in `tierpurity_test.go`.
