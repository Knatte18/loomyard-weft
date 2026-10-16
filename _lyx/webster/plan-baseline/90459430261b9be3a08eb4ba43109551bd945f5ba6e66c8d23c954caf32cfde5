# Card 1 — Declare the LYX_BIN key in agentname

**Create:**
- `plan:internal/agentname#LyxBinEnv` -> `const LyxBinEnv = "LYX_BIN"`

**Delete:**
- `internal/reedengine#lyxBinEnvKey`

**Edit:**
- `internal/reedengine#paneBinPrelude`
- `internal/reedengine#TestPaneBinPrelude_ComposesPrependThenExport`
- `internal/reedengine#TestComposePaneLaunchLine`

**Intent:**
D6: the `LYX_BIN` key's one declaration moves from reedengine's unexported `lyxBinEnvKey` to an exported `LyxBinEnv` in `internal/agentname`, in the existing `const` block beside `StrandNameEnv` and `ParentEnv`.
Its comment says it is the env key reed's launch script exports with the absolute path of the spawning `lyx` binary, never its directory, and that the Claude engine's env file reads the same key.
The constant is an untyped string literal, so the leaf's stdlib-only import set (`TestAgentNameInvariant_StdlibOnly`, `PATTERN-leaf-packages`) is unchanged.

`paneBinPrelude` exports `agentname.LyxBinEnv` instead of the deleted constant; `panebin.go` already imports `agentname`.
The two tests that named `lyxBinEnvKey` name `agentname.LyxBinEnv`; their assertions are otherwise unchanged and keep pinning the prelude's export of the key.
This is a pure move: no behavior changes, and the existing prelude tests are its coverage.

**ImpactSummary:** One unexported constant replaced by an exported one; its sole production reader is `paneBinPrelude` and its only test readers are the two panebin tests, all in `internal/reedengine`.

**Commit:** 1: Declare the LYX_BIN env key in agentname beside the other strand env keys
