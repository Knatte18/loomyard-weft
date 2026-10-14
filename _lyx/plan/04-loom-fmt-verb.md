# Card 4 — loom-fmt-verb

**Create:**
- `internal/loomcli/fmt.go`
- `internal/loomcli/fmt_test.go`
- `plan:internal/loomcli#loomCLI.fmtCmd` -> `func (c *loomCLI) fmtCmd() *cobra.Command`
- `plan:internal/loomcli#fmtResult` -> `type fmtResult struct`
- `plan:internal/loomcli#formatGoFiles` -> `func formatGoFiles(root, cwd string, paths []string) (fmtResult, error)`
- `plan:internal/loomcli#TestFmtCmd` -> `func TestFmtCmd(t *testing.T)`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli/doc.go`
- `contracts/stencils/edit/edit-directive.md`
- `internal/editdirective#TestDirective_RendersTheRuleWithoutBannerOrMarkers`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decision D6: `lyx loom fmt <file>...` rewrites each named `.go` file in place to its `go/format` form, in-process, with no `gofmt` binary, and the edit directive names it.

`fmtCmd` builds the cobra subcommand `fmt <file>...`, audience `operator` beside `lint-comments` so `lyx help index` lists it, with `Args: cobra.MinimumNArgs(1)` and a `Short`/`Long`/example in the form `cmd/lyx/clitree_test.go` pins.
Its root is `c.env.WorktreeRoot`, and it resolves a relative argument against `lyxcwd.CwdFrom(cmd.Context())`, so it calls neither `os.Getwd` nor `git rev-parse` (`PATTERN-cwd-resolution`).
`Command` registers it on the loom parent, and `verbUsesLightweightWiring` admits `"fmt"`, since it reads only `c.env`'s path fields; that function's doc comment names it beside the two lints.

`formatGoFiles` does the work, given the root, the cwd and the argument paths:
1. It resolves every path and the root the same way: joined onto cwd when relative, then `filepath.EvalSymlinks`, so a symlink or junction resolves before the containment check.
2. It checks every file before writing any: a path whose resolved form is not strictly inside the resolved root, a path not ending in `.go`, a path that cannot be read and a file whose content `go/format.Source` cannot format are each refused, named with the reason.
   When any file is refused, it writes nothing and returns the refusals in `fmtResult`.
3. Otherwise it writes, in argument order, only the files whose formatted content differs, through `os.WriteFile` with the file's own mode from its `Lstat` of the resolved path, so the mode is kept.
   A write that fails stops the call: `fmtResult` names the files already written and the one that failed, with its error.
`fmtResult` carries `Changed []string`, `Refused []string` (each `path: reason`) and `Failed string`.

Envelopes: success is `output.Ok` with `changed` (possibly empty), so a formatted file is left untouched and unreported.
A refusal is `output.ErrFields` with message `loom: fmt refused; nothing was written` and `refused`, exit nonzero.
A write failure is `output.ErrFields` with message naming the failed file and error, with `written` and `failed`, exit nonzero.
No argument is cobra's own arity refusal.
Bound: the verb rewrites only `.go` files the caller names inside its own worktree, and only to their `go/format` form; it runs no subprocess and reads no config.

`contracts/specs/refusal-spec.md`'s `## loom` section gains rows, beside the card verbs' rows, for: fmt naming no file (way forward: name at least one `.go` file), fmt refusing a file outside the worktree, not `.go`, unreadable or unparsable (way forward: fix or drop each path the `refused` list names and re-run; nothing was written), and fmt failing a write after earlier files were written (way forward: fix the failed file's permissions and re-run, the written files are already formatted).

`contracts/stencils/edit/edit-directive.md` gains one line after the `sed -n` line: "A Go file's formatting is fixed with `lyx loom fmt <file>`."
`TestDirective_RendersTheRuleWithoutBannerOrMarkers` adds "`lyx loom fmt <file>`" to its wanted phrases, which pins that the rendered directive names the verb.

`TestFmtCmd` is a new top-level, untagged table test in `fmt_test.go`, driving `c.fmtCmd()` through `clihelp.ExecuteIn` with `c := &loomCLI{env: shedrecipe.Env{WorktreeRoot: root}}` over files under `t.TempDir()`, spawning nothing.
Its rows:
- a misformatted `.go` file is rewritten to `go/format`'s form and listed in `changed`; a formatted one beside it is byte-identical afterwards and not listed;
- a rewritten file created with mode `0o600` still has mode `0o600`;
- a relative argument resolves against the injected cwd;
- a call naming a `.txt` path, a path outside the root, a symlink inside the root resolving outside it (skipped where `os.Symlink` fails) or an unparsable `.go` file, each beside a misformatted good file, refuses with exit nonzero, the `refused` list naming each refused path, and the good file unchanged;
- no argument refuses with exit nonzero;
- a misformatted file followed by a misformatted file with mode `0o444` writes the first, then fails: exit nonzero, `written` names the first, `failed` names the second (skipped when running as root, where the mode does not block the write).
- the root reached through a symlink (the root argument a symlink to the real temp directory, skipped where `os.Symlink` fails), with a misformatted file named through the real path, is accepted and rewritten, which fails if the root is not resolved the same way as the paths;
- a call naming a `.go` path inside the root that does not exist, beside a misformatted good file, refuses as "cannot be read", names that path in `refused` and writes nothing; step 1's `filepath.EvalSymlinks` failure on such a path is reported as this refusal, not as an internal error.
`TestVerbUsesLightweightWiring` gains the row `{"Fmt", "fmt", true}`.
`loomCLI.resolveRunID` treats `fmt`'s positional arguments as files, not a run-id: `"fmt"` joins `reject`, `insert-card` and `remove-card` in its exception and in the comment above it, so `lyx loom fmt a.go` keeps `shedrun.SelfRunID`.
`TestResolveRunID`'s table in `arm_seed_test.go` gains the row "fmt's file is not a run-id" with `verb: "fmt"`, `args: []string{"a.go"}` and `wantRunID: shedrun.SelfRunID`, which fails while `resolveRunID` reads the file as a run-id.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/loomcli` from 221 to 222.

`docs/overview.md` adds `fmt` to the loom verb list and a bullet beside `lint-comments [--base <commit>]` describing it; `internal/loomcli/doc.go` names the verb where it describes the lint verbs, or adds a sentence if it names none.
The verb is not in any agent settings deny list, so the directive's line names a step the agents may run (`PATTERN-no-denied-recovery`).

**ImpactSummary:** Adds one loom verb in a new file, registered in `Command` and admitted to the lightweight wiring set; one new line in the shared edit directive reaches every spawned role; one refusal-spec block, one budget row, two doc updates.
