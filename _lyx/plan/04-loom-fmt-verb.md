# Card 4 — `lyx loom fmt` and the edit directive line naming it

**Create:**
- `plan:internal/loomcli#loomCLI.fmtCmd` -> `func (c *loomCLI) fmtCmd() *cobra.Command`
- `plan:internal/loomcli#fmtOutcome` -> `type fmtOutcome struct`
- `plan:internal/loomcli#formatGoFiles` -> `func formatGoFiles(root, cwd string, paths []string) fmtOutcome`
- `plan:internal/loomcli#TestFmtCmd` -> `func TestFmtCmd(t *testing.T)`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli/doc.go`
- `internal/editdirective#TestDirective_RendersTheRuleWithoutBannerOrMarkers`
- `contracts/stencils/edit/edit-directive.md`
- `cmd/lyx/testdata/test-budget.yaml`
- `docs/overview.md`

**Intent:** D6, under `PATTERN-cli-cobra`, `PATTERN-cwd-resolution`, `PATTERN-edit-directive` and `PATTERN-no-denied-recovery`.

The verb: new file `internal/loomcli/fmt.go` holds `fmtCmd`, `fmtOutcome` and `formatGoFiles`.
`fmtCmd` is `Use: "fmt <file>..."`, a one-line lower-case `Short` ("rewrite the named Go files to their gofmt form"), a `Long` with an example, `Args: cobra.MinimumNArgs(1)`, `clihelp.AudienceOperator` like `lint-comments`, so `lyx help index` (the operator command index the `ly:cli` skill renders) lists it, and `RunE` checking `clihelp.ShouldAbort` first.
It reads `c.env.WorktreeRoot` and `c.cwd`, which the lightweight wiring fills from `lyxcwd`, so it resolves no cwd itself.
`formatGoFiles` joins each relative path onto `cwd`, resolves the path and `root` alike with `filepath.EvalSymlinks` (symlinks and junctions included), and refuses a path whose resolved form is not strictly under the resolved root, a path not ending in `.go`, a path that cannot be read and a file `go/format.Source` cannot parse, collecting every refusal with its file and reason.
It checks and formats every file before writing any: one refusal refuses the whole call with nothing written.
It writes a file only when the formatted bytes differ, in place over the existing file so its mode is kept, and records each written file as changed; an unchanged file is neither written nor reported.
A write failure stops the call at that file and records the files already written and the failed one.
`fmtCmd` renders the outcome as one envelope: success is `output.Ok` with `changed` (the written files, empty when none); refusal is `output.ErrFields` with `refused` naming each refused file and its reason; a write failure is `output.ErrFields` with `written` and `failed`; both exit nonzero.
No `gofmt` binary is run; formatting is in-process `go/format`.
Bound: the verb writes only `.go` files the caller names, resolved inside its own worktree, and only their `go/format` form; it is not on any deny list in `claudeengine`'s settings (no `go` subcommand, no `sed`/`perl`, no computed `rm`), so the directive line names an allowed step.

Wiring: `Command` (`internal/loomcli/cli.go`) adds `c.fmtCmd()` to the loom subcommands.
`verbUsesLightweightWiring` adds `"fmt"`, since the verb needs only the resolved paths and no module config; `TestVerbUsesLightweightWiring` gets the row `{"Fmt", "fmt", true}`.
`resolveRunID` (`internal/loomcli/arm.go`) adds `"fmt"` to the verbs whose positional arguments are not a run-id, because otherwise `lyx loom fmt internal/x/a.go` arms `internal/x/a.go` as a run-id and `shedrun.ReadSeed`'s `ValidateRunID` refuses it in the pre-run; `TestResolveRunID` gets the row `{name: "fmt's file paths are not a run-id", verb: "fmt", args: []string{"a.go"}, wantRunID: shedrun.SelfRunID}`, which fails while `fmt` is missing from that list.

`TestFmtCmd` is a new Tier-1 test in a new file `internal/loomcli/fmt_test.go` (package `loomcli`, untagged, spawns nothing): it builds `&loomCLI{env: shedrecipe.Env{WorktreeRoot: root}, cwd: root}` over a `t.TempDir()` root, runs `c.fmtCmd()` through `clihelp.Execute` and decodes the envelope with `envelope.Decode`, as `lintcomments_integration_test.go` does.
Rows, each asserting exit code, envelope and file bytes on disk:
- a misformatted `.go` file is rewritten to its `format.Source` bytes and listed in `changed`;
- a formatted file beside it is left byte-identical, its modification time unchanged, and not listed;
- a misformatted file at mode `0o600` keeps mode `0o600` after the rewrite (skipped on Windows);
- naming no file refuses with a nonzero exit (cobra's argument check);
- one call naming a misformatted file together with a `.txt` path, a path outside the root, a symlink inside the root resolving to a `.go` file outside it (skipped where symlinks are unavailable) and an unparsable `.go` file refuses with a nonzero exit, `refused` naming exactly those four bad files, each with its reason, and the misformatted file left unwritten, so a refusal that stops at the first bad file fails the row;
- a root reached through a symlink (`WorktreeRoot` and `cwd` both the link to a `t.TempDir()`, skipped where symlinks are unavailable) with a misformatted file under it named by a relative path is rewritten and listed in `changed`, not refused, so a root left unresolved beside a resolved path fails the row;
- a call naming two misformatted files, the second made read-only (`0o444`, skipped on Windows and as root), writes the first, fails on the second, and the envelope's `written` names the first and `failed` the second.
`cmd/lyx/testdata/test-budget.yaml`: raise `internal/loomcli` from 221 to 222 for `TestFmtCmd`.

Directive: `contracts/stencils/edit/edit-directive.md` gains one line after the `sed -n` line: "A Go file's formatting is fixed with `lyx loom fmt <file>`."
It still renders through `internal/editdirective`, so every role's opening stencil carries it.
`TestDirective_RendersTheRuleWithoutBannerOrMarkers` adds "`lyx loom fmt`" to its wanted phrases, so the rendered directive must name the verb.

Docs: `internal/loomcli/doc.go` gains a sentence on `fmt`; `docs/overview.md` adds `fmt` to the loom verb list and a line under the `lint-pattern` line saying `fmt <file>...` rewrites the named `.go` files inside the worktree to their `go/format` form, all-or-nothing, so an agent fixes what `cmd/lyx/gofmt_test.go` fails without a rewriting tool.

**ImpactSummary:** Adds one loom verb in loomcli plus two one-word list edits on its pre-run path, each pinned by its table test; one stencil line reaches every spawned role through editdirective; no other package changes.

**Commit:** 4: loom: add lyx loom fmt and name it in the edit directive
