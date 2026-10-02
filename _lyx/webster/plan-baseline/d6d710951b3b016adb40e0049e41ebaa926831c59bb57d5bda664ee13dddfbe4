# Card 2 — fabric-code-record

**Create:**
- `internal/fabricengine/codebinding.go`
- `internal/fabricengine/codebinding_test.go`
- `internal/fabricengine/codebinding_clone_integration_test.go`

**Edit:**
- `internal/fabricengine/warpprobe.go`
- `internal/fabricengine/clone.go`
- `internal/fabriccli/clone.go`
- `internal/fabriccli/fabric.go`
- `internal/hubforge/hub.go`
- `internal/hubforge/hub_test.go`
- `tools/sandbox/main.go`
- `internal/fabriccli/cli_test.go`
- `internal/fabricengine/clone_adopt_test.go`
- `internal/fabricengine/clone_emptyweft_integration_test.go`
- `internal/fabricengine/clone_explicit_root_subpath_test.go`
- `internal/fabricengine/clone_test.go`
- `internal/fabricengine/destructivegaps_integration_test.go`
- `internal/fabricengine/hubcontainment_integration_test.go`
- `internal/fabricengine/hubscratch_integration_test.go`
- `internal/fabricengine/livestate_refusal_selftest_test.go`
- `internal/fabricengine/livestate_verbs_test.go`
- `internal/fabricengine/warpbinding_clone_integration_test.go`
- `internal/fabricengine/warpbinding_reconcile_integration_test.go`

**Uses:**
- `internal/agentname/agentname.go`
- `internal/fabricengine/warpbinding.go`

**Intent:**
Record each repo's short code as a single-line file `.lyx-code` at the board root, committed on weft:main beside `.lyx-warp`, and make `lyx fabric clone` take it.

`codebinding.go` mirrors `warpbinding.go`: an exported `CodeFileName = ".lyx-code"`, an exported `ReadCode(boardDir string) (code string, found bool)` (any read error and an empty-after-trim value report not found, like `readWarpBinding`; a value failing `agentname.ValidateCode` also reports not found and logs a named `logger.Warn` with the value, so a damaged record reads as an uncoded hub and `lyx fabric code <code>` or `clone --code` records over it), an exported `WriteCode(boardDir, code string) error` writing the code plus a newline, and the pure, git-free rule `resolveEffectiveCode(recorded string, found bool, supplied string, freshBind bool) (effective string, writeRecord bool, warning string, err error)`:
- fresh bind, no `--code`: refuse, naming `--code <code>` and the grammar;
- record present, no `--code` or an equal one: derive it, write nothing;
- record present, a differing `--code`: refuse, naming the recorded code (`lyx fabric code` cannot change it either);
- bound weft without a record, `--code` given: record it;
- bound weft without a record, no `--code`: succeed, and return a warning naming `lyx fabric code <code>`.
A supplied code is validated with `agentname.ValidateCode` before anything else.
Both writers stay off git: the CLI layer commits through Bolt, as for `.lyx-warp` (Fabric Git Invariant).

`probeWeftBinding` also reports the code recorded at the probed HEAD (`RecordedCode`, `CodeFound`) and whether the anchor marker is present (`AnchorFound`), read with the same `probeTreeHasPath` + `git show HEAD:` pair it uses for `.lyx-warp`;
the code value goes through the same blank-and-grammar rule as `ReadCode`, so a malformed one reports `CodeFound` false.
A fresh bind is a probe with neither the warp binding nor the anchor marker: an unborn or empty weft that lyx has never touched.
A weft carrying either is bound.

`CloneOptions` gains `Code string`; `CloneResult` gains `Code` (the effective code, empty when none), `CodeRecorded bool` and `Warning string`.
`CloneHub` runs `resolveEffectiveCode` right after `resolveEffectiveWarpURL` in both argument forms, so a refusal fires before the hub directory exists, and writes the record beside the warp binding when told to, appending a `KindFileWritten` mutation.
`--reset` needs no special case: the record lives on the weft remote, so the re-clone derives it.

`fabriccli`: the clone command gains `--code <code>` (help text names the grammar), passes it through `runCloneWithReset` into `CloneOptions.Code`, and the envelope gains `code`, `code_recorded`, and `warning` (always present, empty when none).
`CloneAndWire`'s existing Bolt commit already picks up the new file.

Fixtures: `hubforge.NewHub` passes a fixed test code, so every hub fixture is coded (hubforge Fabric-Fixture Invariant).
`tools/sandbox/main.go`'s fresh clone passes `--code`.
Every listed engine and CLI test whose clone binds a fresh weft passes a code; a clone of an already-bound weft keeps passing none.

Tests: `codebinding_test.go` covers `ReadCode`/`WriteCode` round trip, absent and blank records, a record failing the grammar (read as not found), and every row of `resolveEffectiveCode`, asserting the exact refusal and warning texts name `--code` and `lyx fabric code`.
`codebinding_clone_integration_test.go`: a fresh bind without a code refuses naming the flag and leaves no hub;
a fresh bind with a code commits `.lyx-code` on weft:main;
a second clone of that weft derives the code;
a disagreeing code refuses naming the recorded one;
a bound weft with no record accepts a code and records it, and without one succeeds with the warning;
`--reset` re-clone keeps the record.

**ImpactSummary:** Adds a required code to fresh fabric clones; every caller of CloneHub/CloneAndWire that binds a fresh weft (hubforge, the sandbox tool, fabric tests) passes one.

**Commit:** 2: fabric clone: record the repo's short code as .lyx-code
