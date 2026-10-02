# Card 18 — fabric-shortname

**Move:**
- `internal/fabricengine/codebinding.go`
- `internal/fabricengine/codecommit.go`
- `internal/fabricengine/codebinding_test.go`
- `internal/fabricengine/codebinding_clone_integration_test.go`
- `internal/fabriccli/code.go`
- `internal/fabriccli/code_integration_test.go`

**Edit:**
- `internal/fabricengine/clone.go`
- `internal/fabricengine/warpprobe.go`
- `internal/fabriccli/clone.go`
- `internal/fabriccli/fabric.go`
- `internal/hubforge/hub.go`
- `internal/hubgeom/hubgeom.go`
- `internal/reedengine/strand.go`
- `tools/sandbox/main.go`
- `internal/fabricengine/clone_adopt_test.go`
- `internal/fabricengine/clone_emptyweft_integration_test.go`
- `internal/fabricengine/clone_test.go`
- `internal/fabricengine/destructivegaps_integration_test.go`
- `internal/fabricengine/hubcontainment_integration_test.go`
- `internal/fabricengine/hubscratch_integration_test.go`
- `internal/fabricengine/warpbinding_clone_integration_test.go`
- `internal/fabricengine/warpbinding_reconcile_integration_test.go`
- `internal/fabriccli/cli_test.go`
- `internal/fabriccli/argsarity_test.go`
- `internal/hubforge/hub_test.go`
- `internal/hubgeom/reedgeom_integration_test.go`
- `internal/reedengine/strand_test.go`
- `internal/reedcli/list_integration_test.go`
- `internal/reedcli/naming_integration_test.go`
- `cmd/lyx/helptree_test.go`
- `cmd/lyx/uncontainedwrite_test.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Intent:**
Rename the repo's short code to "shortname" across its record, its clone flag and its verb, and in every text that names them, with no behaviour change.
The record, the flag, the verb and the texts naming them flip in this one commit, so no commit points a user at a spelling that does not exist.

Relocate the six files with `git mv` first, then edit them in place:
`fabricengine/codebinding.go` to `fabricengine/shortnamebinding.go`, `fabricengine/codecommit.go` to `fabricengine/shortnamecommit.go`, `fabricengine/codebinding_test.go` to `fabricengine/shortnamebinding_test.go`, `fabricengine/codebinding_clone_integration_test.go` to `fabricengine/shortnamebinding_clone_integration_test.go`, `fabriccli/code.go` to `fabriccli/shortname.go`, and `fabriccli/code_integration_test.go` to `fabriccli/shortname_integration_test.go`.
Each moved file's leading file comment names its new file name.

The record: `CodeFileName` becomes `ShortnameFileName` with the value `.lyx-shortname`; it stays a single-line file at the board root on weft:main beside `.lyx-warp`.
`ReadCode`, `WriteCode`, `CommitCode`, `usableCode` and `resolveEffectiveCode` become `ReadShortname`, `WriteShortname`, `CommitShortname`, `usableShortname` and `resolveEffectiveShortname`;
`CloneOptions.Code`, `CloneResult.Code` and `CloneResult.CodeRecorded` become `Shortname` and `ShortnameRecorded`;
`warpProbeResult`'s `RecordedCode` and `CodeFound` become `RecordedShortname` and `ShortnameFound`.
Locals, parameters and doc comments follow (`effectiveCode`, `writeCode`, `codeWarning`, `codePresent` and the rest), and each renamed symbol's doc comment opens with its new name.

The CLI: the verb `lyx fabric code [<code>]` becomes `lyx fabric shortname [<shortname>]`, with its `Use`, `Short`, `Long` and examples rewritten, and `runCode` becomes `runShortname`.
The clone flag `--code` becomes `--shortname`, and `runCloneWithReset`'s `code` parameter becomes `shortname`.
The JSON envelope keys follow the concept: the clone envelope's `code` and `code_recorded` become `shortname` and `shortname_recorded`, and the verb's `code` key becomes `shortname`; `clone.go`'s envelope comment says so.

Every refusal, warning and help text that names the record, the flag or the verb names the new spelling: the clone refusals and the bound-weft warning in the moved `shortnamebinding.go`, the verb's refusals in the moved `shortname.go`, the logged warning for an unusable record, the flag help in `fabric.go`, and reed's spawn refusal in `reedengine/strand.go`, which becomes `no strand name can be formed: this hub records no shortname; way forward: lyx fabric shortname <shortname> records it, then retry`.
"uncoded" in comments, texts and test names becomes a "has no shortname" wording, and "coded" a "has a shortname" one.

Fixtures and tools: `hubforge.TestCode` becomes `TestShortname` (value `tst` unchanged), and its callers follow;
`tools/sandbox/main.go`'s `sandboxCode` becomes `sandboxShortname` and its clone passes `--shortname`;
every fabric test that sets `Code: "tst"` sets `Shortname: "tst"`, and every CLI test that passes `--code tst` passes `--shortname tst`.
Test functions and helpers carrying "Code" for this concept rename it to "Shortname" (`TestCodeVerb_*` to `TestShortnameVerb_*`, `runCodeVerb`, `removeCodeRecord`, `TestCodeRecord_RoundTrip`, `TestReadCode_NotFound`, `TestResolveEffectiveCode`, `TestCloneHub_FreshBindWithoutCodeRefuses`, `TestCloneHub_CodeRecordLifecycle` and their subtests), and assertions that look for `--code`, `lyx fabric code` or `.lyx-code` look for the new spelling.
`argsarity_test.go`'s table key, both help-tree listings and `uncontainedwrite_test.go`'s allowlist entry (keyed on the moved file's path, its reason naming `WriteShortname` and `.lyx-shortname`) follow.
`hubgeom.go` calls `ReadShortname`; its `NameCode` field is card 19's.

`refusal-spec.md`'s two "strand spawn: no short code" rows become "strand spawn: no shortname", naming `.lyx-shortname` and `lyx fabric shortname <shortname>`.

The shortname grammar, the record's place on weft:main and every clone and verb rule stay as they are; the existing tests, renamed and re-spelled, are the check that nothing else moved.

**ImpactSummary:** Renames fabric's shortname record, flag, verb, envelope keys and their engine symbols; callers in hubforge, hubgeom, reedengine's refusal text, reedcli tests, cmd/lyx tests and the sandbox tool follow.

**Commit:** 18: fabric: rename the repo code to shortname — .lyx-shortname, --shortname, lyx fabric shortname
