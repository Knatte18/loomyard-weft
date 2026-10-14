# Card 4 — loom-fmt-verb

**Create:**
- `plan:internal/loomcli#loomCLI.fmtCmd` -> `func (c *loomCLI) fmtCmd() *cobra.Command`
- `plan:internal/loomcli#fmtFile` -> `type fmtFile struct`
- `plan:internal/loomcli#checkFmtFiles` -> `func checkFmtFiles(root, cwd string, paths []string) (files []fmtFile, refused []string)`
- `plan:internal/loomcli#TestFmtCmd` -> `func TestFmtCmd(t *testing.T)`
- `internal/loomcli/fmt.go`
- `internal/loomcli/fmt_test.go`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli/doc.go`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D6: add `lyx loom fmt <file>...`, which rewrites each named `.go` file in place to its `go/format` form, in-process, with no `gofmt` binary.
The verb and its helpers live in a new `internal/loomcli/fmt.go`.

`fmtCmd` is a cobra subcommand (`Use: "fmt <file>..."`, audience `clihelp.AudienceOperator`, so `lyx help index`, the operator command index, lists it), registered in `Command`'s `AddCommand` list beside `lintCommentsCmd`, with `cobra.ArbitraryArgs` and the `clihelp.ShouldAbort` guard the other loom verbs use.
The root is `c.env.WorktreeRoot`, which the lightweight wiring resolves through `lyxcwd` (`PATTERN-cwd-resolution`, `PATTERN-cli-cobra`), and a relative argument is joined to the cwd from `lyxcwd.CwdFrom(cmd.Context())`.
Add `"fmt"` to `verbUsesLightweightWiring`, so a broken module config never refuses it, and to `resolveRunID`'s list of verbs whose positional arguments are not a run-id (beside `reject`, `insert-card` and `remove-card`): without it `arm` reads a file path as a run-id and `shedrun.ReadSeed`'s run-id validation refuses every call.
The `verbUsesLightweightWiring` doc comment names fmt among the verbs that read only `c.env`'s path fields.

`checkFmtFiles` resolves the root and each path with `filepath.EvalSymlinks` (the same way, so symlinks and junctions on either side compare alike), and refuses a path that does not resolve, whose resolved form does not end in `.go`, whose resolved form is not inside the resolved root (`filepath.Rel` starting with `..`), or whose content `go/format.Source` cannot format.
Each refusal is one string naming the path as given and the reason.
For an accepted path it returns a `fmtFile` holding the resolved path, the formatted bytes, the file's mode and whether the bytes differ from the content read.
The verb runs `checkFmtFiles` over every argument before writing anything:
- No argument refuses with `output.Err` naming that at least one `.go` file is required, exit 1, nothing written.
- Any refusal refuses the whole call with `output.ErrFields` and a `refused` list of the refusal strings, exit 1, nothing written.
- Otherwise it writes each file whose bytes differ, with `os.WriteFile` and the file's own permission bits, so the mode is kept, and an unchanged file is neither written nor reported.
- A write failure stops at that file and returns `output.ErrFields` with `written` (the files written before it) and `failed` (the file and its error), exit 1.
- Success returns `output.Ok` with `changed`, the paths it rewrote, always an array.

Bound: the verb writes only `.go` files the caller names, resolved inside the caller's own worktree, and only their `go/format` form, which parses the file first; it shells out to nothing and is not denied by any agent settings (`PATTERN-no-denied-recovery`).

Tests: `TestFmtCmd` in the new `fmt_test.go`, Tier 1 (a temp directory as `c.env.WorktreeRoot`, no git; `c := &loomCLI{env: shedrecipe.Env{WorktreeRoot: dir}}` driven through `clihelp.Execute` and decoded with `envelope.Decode`, as `lintcomments_integration_test.go` drives its verb), parallel, one table, absolute paths unless a row says otherwise:
- a misformatted file at mode 0o600 is rewritten to `format.Source`'s bytes, keeps mode 0o600 and is listed in `changed`;
- a formatted file is untouched (bytes and modification time unchanged) and not listed;
- a call naming a misformatted file beside a non-`.go` file refuses, names the non-`.go` file in `refused`, and leaves the misformatted file unchanged;
- the same for a `.go` file outside the root (in a second temp directory), a symlink inside the root resolving to a `.go` file outside it (skipped on Windows), and a `.go` file that does not parse;
- no argument refuses, exit 1;
- a relative path: driven through `clihelp.ExecuteIn` with a subdirectory of the root as the cwd and a misformatted file in that subdirectory named by its bare file name, the file is rewritten, so a join against the process cwd or the root would fail the row;
- a write failure: two misformatted files, the second chmod 0o444 (skipped on Windows and as root), exits 1 with the first in `written`, the second in `failed`, and the first rewritten.

`TestVerbUsesLightweightWiring` gains the row `{"Fmt", "fmt", true}`, and `TestResolveRunID` the row `fmt's files are not a run-id` (verb `fmt`, args `[]string{"a.go"}`, `wantRunID: shedrun.SelfRunID`).
The new top-level test raises `internal/loomcli`'s row in `cmd/lyx/testdata/test-budget.yaml` from 221 to 222.

Docs: `internal/loomcli/doc.go` gains a sentence naming `lyx loom fmt` as the agent-facing formatter beside `lint-comments`, and `docs/overview.md` adds `fmt` to the loom verb list in the module table line and a `fmt <file>...` sentence after the `lint-comments` sentence of the loom section.

**ImpactSummary:** One new verb in loomcli with its own file and test; the two verb-set functions gain one name each, which their two table tests pin; no existing verb's behavior changes.

**Commit:** 4: Add lyx loom fmt to format named Go files in place
