# Card 4 — loom-fmt-verb

**Create:**
- `internal/loomcli/fmt.go`
- `plan:internal/loomcli#loomCLI.fmtCmd` -> `func (c *loomCLI) fmtCmd() *cobra.Command`
- `plan:internal/loomcli#formatGoFiles` -> `func formatGoFiles(root, cwd string, paths []string) (fmtOutcome, error)`
- `plan:internal/loomcli#fmtOutcome` -> `type fmtOutcome struct`
- `internal/loomcli/fmt_test.go`
- `plan:internal/loomcli#TestFmtCmd_FormatsNamedGoFiles` -> `func TestFmtCmd_FormatsNamedGoFiles(t *testing.T)`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli/doc.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/loomcli#loomCLI.lintCommentsCmd`

**Intent:** D6: a `lyx loom fmt <file>...` verb that rewrites named Go files to their `go/format` form in-process, so an agent the edit directive bars from every rewriting tool has a sanctioned way to fix what `cmd/lyx/gofmt_test.go` fails.
It lives in `loomcli` beside `lint-comments`, the other agent-facing Go source verb, and follows its shape.

`fmtCmd` builds the cobra command: `Use: "fmt <file>..."`, audience `clihelp.AudienceOperator` (so `lyx help index`, the operator command index the `ly:cli` skill renders, lists it with no further file), `Args: cobra.ArbitraryArgs`, a `Long` text with an example, and a `RunE` that honours `clihelp.ShouldAbort`, reads the cwd through `lyxcwd.CwdFrom(cmd.Context())` and calls `formatGoFiles(c.env.WorktreeRoot, cwd, args)`.
`Command` adds `c.fmtCmd()` to loom's `AddCommand` list.
`verbUsesLightweightWiring` adds `"fmt"`, since the verb reads only `c.env.WorktreeRoot` and never a module config; extend its doc comment's list.
`resolveRunID` adds `"fmt"` to the verbs whose positional arguments are not a run-id (its files), beside `reject`, `insert-card` and `remove-card`, so `lyx loom fmt a.go` never addresses a run named `a.go`; extend its comment.

`formatGoFiles` works in two phases and writes nothing until every named file passed the first.
Check phase, per path in argument order: a relative path is joined to `cwd` (never `os.Getwd`, per `PATTERN-cwd-resolution`); the path and `root` are each resolved through `filepath.EvalSymlinks`, so symlinks and junctions resolve the same way on both sides; the path is refused when its resolved form does not end in `.go`, is not strictly inside the resolved root, cannot be stat'ed or read as a regular file, or its content does not parse (`go/format.Source` returns an error).
Every refused path is collected as `<path>: <reason>`; when any path is refused, the call refuses whole, writes nothing, and the envelope names each refused file.
Write phase, in argument order: a file whose formatted bytes equal its content is left untouched and unreported; otherwise it is written with `os.WriteFile` at its own `Mode().Perm()`, so its mode is kept, and recorded as changed.
A write that fails stops the call: the outcome names the files already written and the one that failed, and the call exits nonzero.
`fmtOutcome` carries the changed files, the refused entries and the failed write.
What the verb can now do and what bounds it: it rewrites only `.go` files the caller names inside its own worktree, only to their `go/format` form, and never a file outside the root through a link, since both sides are resolved before the containment check.

Envelopes, each refusal naming its way forward per `PATTERN-refusal-way-forward`:
- no file: `output.Err` with `loom: fmt names no file; way forward: name at least one .go file, as lyx loom fmt <file>...`, exit 1.
- refused file: `output.ErrFields` with `loom: fmt refused, nothing written; way forward: name only .go files inside this worktree that parse, then re-run lyx loom fmt` and the field `refused`, the list of `<path>: <reason>`, exit 1.
- failed write: `output.ErrFields` with `loom: fmt wrote <n> file(s) then failed to write <path>: <err>; way forward: fix the named file's write error, then re-run lyx loom fmt on it` and the fields `written` and `failed`, exit 1.
- success: `output.Ok` with `changed`, the list of rewritten paths as named (empty when none changed).

`refusal-spec.md`'s loom section gains three rows quoting those way-forward clauses: "fmt: no file" (correctness halt), "fmt: file refused" (correctness halt; trigger: a named path is not a `.go` file, resolves outside the worktree, symlinks and junctions followed, is not a readable regular file, or does not parse) and "fmt: write failed partway" (transient; trigger: writing a formatted file fails after earlier files were written).

`TestFmtCmd_FormatsNamedGoFiles` in the new untagged `fmt_test.go` is one table-driven test, each row in its own `t.TempDir()` worktree driven through `clihelp.ExecuteIn(c.fmtCmd(), dir, &out, args)` with `c := &loomCLI{env: shedrecipe.Env{WorktreeRoot: dir}}`, decoding the envelope with `internal/testkit/envelope`.
Rows, each asserting the exit code, the envelope's `ok` and its named field exactly, and every file's bytes after the call:
- a misformatted file named relatively (`a.go`) is rewritten to its `go/format.Source` bytes and listed in `changed`;
- a formatted file is untouched (bytes and modification time unchanged) and `changed` is empty;
- a misformatted file at mode `0o600` keeps mode `0o600` after the rewrite (the mode assertion skipped on Windows);
- a misformatted `.go` file named beside a `notes.txt` refuses whole: `refused` names `notes.txt`, and the `.go` file is unchanged;
- a path outside the worktree (a file in a second `t.TempDir()`) is refused and named;
- a `.go` symlink inside the worktree resolving to a misformatted file outside it is refused, and the outside file is unchanged (skipped on Windows, where creating the link needs a privilege);
- a file that does not parse is refused and named, with a misformatted sibling named in the same call left unchanged;
- a call naming both `notes.txt` and an unparsable `.go` file refuses with `refused` listing both entries, in argument order, which fails if the check stopped at the first refusal;
- no file refuses with exit 1 and the no-file message;
- two misformatted files, the second made read-only (`0o444`), stop at the second: exit 1, `written` names the first, `failed` names the second, the first is formatted and the second unchanged (skipped on Windows and when running as root, where the write would not fail).
Extend `TestVerbUsesLightweightWiring` with the row `{"Fmt", "fmt", true}` and `TestResolveRunID` with the row "fmt's files are not a run-id" (`verb: "fmt"`, `args: []string{"a.go"}`, `wantRunID: shedrun.SelfRunID`), which fail if `fmt` left either set.

The new top-level test pushes `internal/loomcli` past its row: raise it in `cmd/lyx/testdata/test-budget.yaml` from 221 to 222.

Docs: `internal/loomcli/doc.go` gains a paragraph on `fmt`: what it rewrites, the two phases, the refusals and the envelope, and that it joins the lightweight wiring.
`docs/overview.md` adds `fmt` to the loom verb list in the module table line and, beside the `lint-comments` and `lint-pattern` lines, a line saying `fmt <file>...` rewrites the named `.go` files of the worktree to their `go/format` form, in-process, refusing the whole call on a non-`.go`, outside or unparsable file.

**ImpactSummary:** a new loom verb and its test; loom's verb registration, lightweight set and run-id resolution gain `fmt`; refusal spec, loomcli doc, overview and the loomcli budget row follow.

**Commit:** 4: Add lyx loom fmt, an in-process go/format rewrite of named files
