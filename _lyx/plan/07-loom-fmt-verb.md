# Card 7 — loom-fmt-verb

**Create:**
- `internal/loomcli/fmt.go`
- `plan:internal/loomcli#loomCLI.fmtCmd` -> `func (c *loomCLI) fmtCmd() *cobra.Command`
- `plan:internal/loomcli#fmtOutcome` -> `type fmtOutcome struct`
- `plan:internal/loomcli#formatGoFiles` -> `func formatGoFiles(worktreeRoot, base string, paths []string) (fmtOutcome, error)`
- `internal/loomcli/fmt_test.go`
- `plan:internal/loomcli#TestFmtCmd` -> `func TestFmtCmd(t *testing.T)`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli/doc.go`
- `//docs/overview.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Add `lyx loom fmt <file>...` (D6): it rewrites each named `.go` file in place to its `go/format` form, in-process, with no `gofmt` binary.

`fmtCmd` is a cobra leaf beside `lint-comments`: `Use: "fmt <file>..."`, a lower-case `Short`, a `Long` with an example, `Args: cobra.MinimumNArgs(1)`, audience `clihelp.AudienceOperator` so the operator command index (`lyx help index`) lists it, and `clihelp.ShouldAbort` checked first.
It is mounted in `Command`'s `AddCommand` list, joins `verbUsesLightweightWiring`'s set (it reads only `c.env.WorktreeRoot` and `c.env.AnchorPath`), and `resolveRunID` reads no run-id from its arguments, so a file path is never taken as a run-id; the root comes from `lyxcwd` through arming (`PATTERN-cwd-resolution`).
`verbUsesLightweightWiring`'s doc comment names the verb with the lints.

`formatGoFiles` takes the worktree root and the base a relative path resolves against (`c.env.AnchorPath`, which equals the resolved cwd).
For every named path, before writing anything, it joins a relative path onto the base, resolves the path and the root with `filepath.EvalSymlinks` (symlinks and junctions alike), and refuses the path when it is outside the root, does not end in `.go`, cannot be read, or does not parse under `go/format.Source`; each refusal is a `path: reason` string.
Any refusal refuses the whole call with nothing written: `fmtCmd` prints an error envelope listing every refused file under `refused` and exits nonzero.
With no refusal it writes, in argument order, only the files whose formatted bytes differ, through `os.WriteFile` with the file's current mode, so the mode is kept; an unchanged file is neither written nor reported.
`fmtOutcome` holds `Written []string` and `Refused []string`.
A write that fails returns the files already written and an error naming the failing file; `fmtCmd` prints an error envelope with `written` and `failed` and exits nonzero.
Success prints an ok envelope with `changed`, the written files (empty when none changed).
Bound: the verb rewrites only `.go` files the caller names inside its own worktree, only to their `go/format` form, and refuses before writing on any doubt; it is not denied by any agent setting, so the edit directive may name it (`PATTERN-no-denied-recovery`).

Tests: `TestFmtCmd` (untagged, `t.TempDir` worktree, no git) drives `c.fmtCmd()` through `clihelp.Execute` on a `loomCLI` whose `env` carries `WorktreeRoot` and `AnchorPath`, with rows:
- a misformatted file is rewritten to its `go/format` form and reported in `changed`, its mode (0o640) kept;
- a formatted file is left byte-identical, its modification time unchanged, and not reported;
- a call naming a misformatted file beside a non-`.go` path refuses, names the non-`.go` path under `refused`, and leaves the misformatted file unwritten;
- the same for a path outside the worktree, a symlink inside the worktree resolving to a `.go` file outside it (skipped where a symlink cannot be made), and an unparsable `.go` file;
- a call naming a non-`.go` path and a path outside the worktree together lists both under `refused`, so a refusal that stops at the first refused file fails;
- a worktree root told through a symlink to the temporary directory, with a misformatted file named by its real path, is rewritten, so a root left unresolved, which would read the file as outside, fails (skipped where a symlink cannot be made);
- naming no file refuses with a nonzero exit;
- a write failure (the second named file made read-only, skipped when running as root) exits nonzero with the first file under `written` and the second under `failed`.
`TestVerbUsesLightweightWiring` gains `{"Fmt", "fmt", true}`, and `TestResolveRunID` gains `fmt's file is not a run-id` with args `["x.go"]`, `wantRunID: shedrun.SelfRunID`.
`test-budget.yaml`'s `internal/loomcli` row rises by one for `TestFmtCmd`.
`loomcli/doc.go` gains a paragraph on the verb and its bound; `docs/overview.md` adds `fmt` to loom's verb list and a line describing it beside `lint-comments`.

**ImpactSummary:** New loom leaf verb plus one entry each in the lightweight-wiring set, the run-id exclusion and the AddCommand list; no existing verb's behavior changes.
