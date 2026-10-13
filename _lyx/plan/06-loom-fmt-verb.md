# Card 6 — `lyx loom fmt` formats named Go files

**Create:**
- `internal/loomcli/fmtverb.go`
- `plan:internal/loomcli#loomCLI.fmtCmd` -> `func (c *loomCLI) fmtCmd() *cobra.Command`
- `plan:internal/loomcli#goRewrite` -> `type goRewrite struct`
- `plan:internal/loomcli#planGoFormat` -> `func planGoFormat(root, cwd string, paths []string) (rewrites []goRewrite, refused []string)`
- `internal/loomcli/fmtverb_test.go`
- `plan:internal/loomcli#TestFmtCmd` -> `func TestFmtCmd(t *testing.T)`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli/doc.go`
- `internal/editdirective#TestDirective_RendersTheRuleWithoutBannerOrMarkers`
- `contracts/stencils/edit/edit-directive.md`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D6: `lyx loom fmt <file>...` rewrites each named `.go` file in place to its `go/format` form, in-process with no `gofmt` binary, so an agent whose edit directive forbids every rewriting tool has a sanctioned way to fix the formatting `cmd/lyx/gofmt_test.go` fails.
It is a cobra subcommand in `loomcli` beside `lint-comments`, registered in `Command`'s `AddCommand` list, with `Args: cobra.ArbitraryArgs` and the operator audience annotation like `lint-comments`, so `lyx help index`, the operator command index `ly:cli` renders, lists it.
`verbUsesLightweightWiring` admits `fmt`, since the verb reads only `c.env.WorktreeRoot`, which `wireLightweight` fills, and `c.cwd`, the cwd the pre-run read through `lyxcwd.CwdFrom`, per `PATTERN-cwd-resolution`; its doc comment's enumeration of the lightweight set and each verb's reason gains `fmt`, which reads only those two paths.

`planGoFormat` checks and formats every named file before any write.
It resolves the worktree root and each path the same way, `filepath.EvalSymlinks` after joining a relative path onto `cwd`, so symlinks and junctions on either side compare alike, then requires the resolved path to lie strictly inside the resolved root and to end in `.go`.
It reads each passing file, formats it with `go/format.Source`, and returns a `goRewrite` (the resolved path, the path as named, the formatted bytes and whether they differ from the file's) for each, or one `refused` entry `<path as named>: <reason>` per path outside the worktree, a symlink resolving outside it, a non-`.go` path, an unreadable file or one that does not parse.
`fmtCmd`'s `RunE`:
- With no file named, it refuses `loom fmt: name at least one .go file; way forward: run "lyx loom fmt <file>..." with the Go files to format`.
- With any refused entry, it refuses the whole call, writing nothing, with an `ErrFields` envelope whose `refused` key lists every refused entry and whose message ends `way forward: name only .go files inside this worktree that parse, then re-run "lyx loom fmt"`.
- Otherwise it writes each rewrite whose bytes differ with `os.WriteFile`, which keeps the file's mode, leaves an already-formatted file untouched and unreported, and answers `output.Ok` with `changed`, the changed files as named.
- A write that fails stops the call with an `ErrFields` envelope naming `written`, the files already rewritten, and `failed`, the file that failed with its error, and the message ends `way forward: re-run "lyx loom fmt" with the same files once the write error is fixed; a formatted file is left untouched`.
Bound: the verb rewrites only `.go` files the caller names inside its own worktree, and only to their `go/format` form; it runs no binary and touches no other file.
`lyx loom fmt` is not in any agent deny list, per `PATTERN-no-denied-recovery`.

The three refusals join `refusal-spec.md`'s `loom` section as rows: "fmt: no file named" and "fmt: a named file refused" as correctness halts, and "fmt: write failed partway" as transient, each quoting its message's way-forward clause, per `PATTERN-refusal-way-forward`.
The edit directive `contracts/stencils/edit/edit-directive.md` gains one line: a Go file's formatting is fixed with `lyx loom fmt <file>`.
It still renders through `editdirective`, per `PATTERN-edit-directive`.
`internal/loomcli/doc.go` names the verb, and `docs/overview.md` adds `fmt` to the loom verb list and one line describing it beside `lint-comments`.

Tests:
- `TestFmtCmd` in the new `fmtverb_test.go`, untagged, drives `c.fmtCmd()` through `clihelp.Execute` over a temp worktree with `c := &loomCLI{env: shedrecipe.Env{WorktreeRoot: dir}, cwd: dir}`, the two fields the pre-run and `wireLightweight` fill, and decodes the envelope with `envelope.Decode`.
- Its rows:
  - a misformatted file is rewritten to its `go/format` form, keeps its 0o600 mode and is the one `changed` entry;
  - an already-formatted file is left with its bytes and modification time and is not reported;
  - a relative path resolves against `cwd`;
  - no file named refuses;
  - a non-`.go` path, a path outside the worktree, a symlink inside the worktree resolving to a `.go` file outside it, and an unparsable file each refuse the whole call, beside a misformatted file in the same call that is left unwritten, with `refused` naming the refused file; the symlink row skips when the platform refuses to create the link, as Windows does without the privilege;
  - a non-`.go` path and an unparsable file named in one call, beside a misformatted file, refuse with `refused` listing both entries and the misformatted file left unwritten;
  - a misformatted file followed by a second misformatted file made read-only (0o444, still readable for formatting) writes the first and fails on the second, exit 1, with `written` and `failed` naming them; the row skips when run as root, where a read-only file does not stop the write.
- `TestVerbUsesLightweightWiring` gains the row `{"Fmt", "fmt", true}`.
- `TestDirective_RendersTheRuleWithoutBannerOrMarkers`'s wanted phrases gain `` `lyx loom fmt` ``.
- `test-budget.yaml`'s `internal/loomcli` row rises by one for `TestFmtCmd`.

**ImpactSummary:** a new loom verb and its refusals inside `loomcli`, registered in `Command` and the lightweight wiring list; the edit directive every spawned role renders gains one line; docs, the refusal spec and one test-budget row follow.

**Commit:** 6: Add lyx loom fmt and name it in the edit directive
