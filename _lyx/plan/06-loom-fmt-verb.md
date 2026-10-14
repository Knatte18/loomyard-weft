# Card 6 — `lyx loom fmt` formats named Go files

**Create:**
- `internal/loomcli/fmt.go`
- `plan:internal/loomcli#loomCLI.fmtCmd` -> `func (c *loomCLI) fmtCmd() *cobra.Command`
- `plan:internal/loomcli#goFormatPlan` -> `func goFormatPlan(root, cwd string, paths []string) (edits []goFormatEdit, refused []string)`
- `plan:internal/loomcli#goFormatEdit` -> `type goFormatEdit struct`
- `internal/loomcli/fmt_test.go`
- `plan:internal/loomcli#TestFmtCmd` -> `func TestFmtCmd(t *testing.T)`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decision D6: `lyx loom fmt <file>...` rewrites each named `.go` file in place to its `go/format` form, in-process, with no `gofmt` binary.
`cmd/lyx/gofmt_test.go` fails a misformatted file and the edit directive forbids every rewriting tool, so this is the agent's sanctioned way to fix formatting.

`fmt.go` (package `loomcli`, file comment naming the verb):
- `fmtCmd` is a cobra command.
  `Use: "fmt <file>..."`, `Args: cobra.MinimumNArgs(1)`, so naming no file refuses with cobra's usage error, exit nonzero, nothing written.
  It carries the role audience annotation (`clihelp.AudienceRole`) and a `Long` text with an `Example:` block, in `lintCommentsCmd`'s shape.
- Its root is `c.env.WorktreeRoot`.
  Its cwd for relative paths comes from `lyxcwd.CwdFrom(cmd.Context())`, per `PATTERN-cwd-resolution`.
- `goFormatPlan` checks and formats every named file before anything is written.
  Each path is made absolute against `cwd` and resolved with `filepath.EvalSymlinks`; the root is resolved the same way, so symlinks and junctions count on both sides.
  A path is refused when:
  - it does not resolve (missing);
  - its named or resolved form lacks the `.go` suffix;
  - its resolved form is not inside the resolved root;
  - its content does not parse under `go/format.Source`.
  Each refusal is one `"<path as named>: <reason>"` string.
  Two names resolving to one file are formatted once.
  A file whose formatted bytes equal its content yields no edit.
- `goFormatEdit` holds the resolved path, the name as given and the formatted bytes.
- RunE: any refusal fails the whole call with nothing written.
  The envelope is `output.ErrFields` with a `refused` list naming each refused file, and a message ending `way forward: name only .go files inside this worktree that parse, then re-run lyx loom fmt`.
- Otherwise each edit is written with `os.WriteFile` over the existing file with its own `Stat` permission bits, so the file keeps its mode, and only changed files are written.
  The envelope is `output.Ok` with `changed`: the changed files as named, empty when none changed.
- A write that fails after earlier files were written stops the call nonzero with `output.ErrFields`.
  The envelope names `written` (the files already written) and `failed` (the file that failed and its error), and the message ends `way forward: fix the failed file's permissions, then re-run lyx loom fmt`.
- Bound: the verb rewrites only `.go` files the caller names inside its own worktree, and only to their `go/format` form.
  It is not denied by the agents' settings (`PATTERN-no-denied-recovery`).

Wiring (`PATTERN-cli-cobra`):
- `Command` adds `c.fmtCmd()` to the loom group.
- `verbUsesLightweightWiring` admits `"fmt"`, so a broken module config never refuses it; its doc comment names fmt beside the two lints.
- `resolveRunID` treats `fmt`'s positionals as files, not a run-id, beside `reject`, `insert-card` and `remove-card`; its comment says so.

Refusal rows (`PATTERN-refusal-way-forward`), in `refusal-spec.md`'s loom table beside the card-verb rows:
- `fmt: no file`: correctness halt; way forward: name at least one `.go` file.
- `fmt: path refused`: a missing, non-`.go`, outside-the-worktree or symlinked-outside path, or an unparsable file, and nothing written; correctness halt; the refusal's way-forward clause.
- `fmt: write failed partway`: transient; the write-failure way-forward clause.

`docs/overview.md`: the loom module line's verb list gains `fmt`.
A line beside the `lint-comments` description says `fmt <file>...` rewrites the named Go files to their `go/format` form, refuses the whole call over any bad path or unparsable file, and reports the changed files.

Tests:
- `TestFmtCmd` (new file `fmt_test.go`, untagged, `t.Parallel`, temp directories, no git).
  It drives `c.fmtCmd()` through `clihelp.ExecuteIn` with `c := &loomCLI{env: shedrecipe.Env{WorktreeRoot: root}}`.
  It decodes the envelope with `envelope.Decode`.
  Rows:
  - A misformatted file chmodded to `0o600` is rewritten to exactly `format.Source`'s output, keeps the permission bits `Stat` read before the call, and is listed in `changed`.
  - A missing `.go` path and a `foo.go` symlink inside the root pointing at a `.txt` file inside it (skipped where `os.Symlink` fails) each refuse the whole call like the rows below.
  - A formatted file is left byte-identical and is not listed.
  - A relative path resolves against the cwd.
  - Each of these refuses the whole call: a `.txt` path, a path outside the root, a symlink inside the root resolving outside it (skipped where `os.Symlink` fails), and an unparsable `.go` file.
    Each also names a misformatted `.go` file in the same call; the call exits nonzero, its `refused` names the bad path, and the good file stays byte-identical.
  - One call naming both the `.txt` path and the unparsable file lists both in `refused`.
  - No file exits nonzero and writes nothing.
  - A write that fails partway: two misformatted files, the second made read-only, skipped when running as root.
    It exits nonzero, names the first as `written` and the second as `failed`, and leaves the first formatted.
- `TestVerbUsesLightweightWiring` gains `{"Fmt", "fmt", true}`.
- `TestResolveRunID` gains "fmt's files are not a run-id": verb `fmt`, args `["a.go"]`, `wantRunID: shedrun.SelfRunID`.
- `cmd/lyx/testdata/test-budget.yaml`: `internal/loomcli` rises by one for `TestFmtCmd`.

**ImpactSummary:** Adds one loom verb and its refusal rows; the lightweight-wiring set and the run-id exclusion each gain `fmt`, with a row in each one's test; no existing verb's behavior changes.

**Commit:** 6: loom: add lyx loom fmt to format named Go files
