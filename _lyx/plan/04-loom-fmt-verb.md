# Card 4 — loom-fmt-verb

**Create:**
- `plan:internal/loomcli#loomCLI.fmtCmd` -> `func (c *loomCLI) fmtCmd() *cobra.Command`
- `plan:internal/loomcli#goRewrite` -> `type goRewrite struct`
- `plan:internal/loomcli#planGoRewrites` -> `func planGoRewrites(root, cwd string, paths []string) (rewrites []goRewrite, refused []string)`
- `internal/loomcli/gofmt.go`
- `internal/loomcli/gofmt_test.go`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#loomCLI.resolveRunID`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli#TestLintCommentsCmd_OverARepository`
- `internal/loomcli/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Intent:**
D6: `lyx loom fmt <file>...` rewrites each named `.go` file in place to its `go/format` form, in-process, with no `gofmt` binary.
It lives in a new `internal/loomcli/gofmt.go`, beside `lint-comments`, the other agent-facing Go source verb.

Bound: the verb rewrites only `.go` files the caller names, inside its own worktree, and only to their `go/format` form; it can skip nothing and lets no other write through.

`goRewrite` holds one planned write: the path as the caller named it, the resolved file path, the formatted content and the file's permission bits.
`planGoRewrites(root, cwd, paths)` checks and formats every named path before anything is written:
it resolves `root` and each path with `filepath.EvalSymlinks` (a relative path is joined to `cwd` first), so symlinks and Windows junctions resolve the same way on both sides;
it refuses a path that cannot be resolved or read, one whose resolved form is not strictly inside the resolved root, one whose resolved form is not a regular file ending in `.go`, and one `format.Source` cannot parse, each as one `"<path as named>: <reason>"` string;
and it returns a `goRewrite` only for a file whose formatted content differs from its current content, so a formatted file is neither written nor reported.

`fmtCmd` builds the `fmt` subcommand: `Use: "fmt <file>..."`, audience `clihelp.AudienceOperator` so the generated operator command index lists it, `cobra.ArbitraryArgs`, and a `Long` with an example.
Its `RunE` reads the worktree root from `c.env.WorktreeRoot` and the caller's cwd from `c.cwd`, both of which the lightweight wiring fills, and answers through `output`:
- no file named: the refusal `loom: fmt names no file; way forward: name the .go files to format, as lyx loom fmt <file>...`, exit 1;
- any refused path: nothing is written, and the refusal `loom: fmt refused <n> file(s) and wrote nothing; way forward: name only .go files inside this worktree that parse, then re-run lyx loom fmt` carries every refused path under `refused`, exit 1;
- otherwise each rewrite is written in order with `os.WriteFile` and its recorded permission bits, which keeps the file's mode, and the ok envelope lists the changed paths, as named, under `changed` (an empty list when none changed);
- a write that fails stops the call: the refusal `loom: fmt could not write <path>: <error>; way forward: re-run lyx loom fmt <path>` lists the files already written under `written` and the failed one under `failed`, exit 1.
`Command` adds `c.fmtCmd()` to loom's subcommands.

`fmt` joins `verbUsesLightweightWiring`'s set, since it reads only `c.env`'s path fields and `c.cwd` and never a module config; the function's doc comment names it with the two lints.
`resolveRunID` stops reading `fmt`'s first argument as a run-id, as it already does for `reject`, `insert-card` and `remove-card`, so a named file is never looked up as a seed.

Tests: `TestVerbUsesLightweightWiring` gains the row `{"Fmt", "fmt", true}`, and `TestResolveRunID` gains the row `fmt's file is not a run-id` with args `a.go` wanting the self run-id.
A new `internal/loomcli/gofmt_test.go` (package `loomcli`, untagged) holds one table-driven `TestFmtCmd`; each row builds a `loomCLI` with `cwd` and `env.WorktreeRoot` set to its own temp directory, seeds files there, runs `c.fmtCmd()` through `clihelp.Execute` and decodes the envelope with `envelope.Decode`.
Rows:
- a misformatted `a.go` of mode `0o600` is rewritten to exactly `format.Source` of its old content, keeps mode `0o600`, and `changed` is `[a.go]`;
- an already formatted `a.go` keeps its content and modification time and `changed` is empty;
- a call naming a misformatted `a.go`, `notes.txt` and an unparsable `broken.go` refuses, names both `notes.txt` and `broken.go` under `refused`, so a verb that stopped at its first refusal fails, and leaves `a.go` byte-for-byte unchanged;
- a path to a `.go` file in a second temp directory outside the worktree refuses and names it;
- a symlink `link.go` inside the worktree to a `.go` file outside it refuses and names it (skipped on Windows, where file symlinks are not relied on);
- an `a.go` that does not parse refuses and names it;
- naming no file refuses with exit 1;
- a misformatted `a.go` then a misformatted read-only `b.go` (mode `0o444`) writes `a.go`, then fails on `b.go`, with `written` `[a.go]`, `failed` `b.go` and exit 1 (skipped when running as root, where the mode does not stop the write, and on Windows).
The read-only row is the only one reaching the `fmt: write failed partway` row, and it is skipped as root and on Windows; that skip is accepted rather than adding a write seam to `loomCLI` for tests alone, and the refusal still has its reaching row on every non-root Unix run, including the gate's.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/loomcli` from 221 to 222 for `TestFmtCmd`.

`TestFmtCmd` calls `fmtCmd` directly, so it does not reach `Command`'s wiring or loom's pre-run, which arms the verb through `resolveRunID` and `wireLightweight`.
`TestLintCommentsCmd_OverARepository` (integration tier, over a real repository) gains the subtest `fmt through loom's command path rewrites a named file`: it writes a misformatted `fmtme.go`, runs `RunCLIIn(dir, &out, []string{"fmt", "fmtme.go"})`, and wants exit 0, `changed` `[fmtme.go]` and the file in its `go/format` form, so a missing `AddCommand`, a pre-run reading `fmtme.go` as a run-id or seed, or a config-loading wiring fails it.

`contracts/specs/refusal-spec.md`'s loom section gains three rows, each quoting its message's way forward, per `PATTERN-refusal-way-forward`: `fmt: no file named` and `fmt: file refused` as correctness halts, and `fmt: write failed partway` as transient; `TestFmtCmd` reaches all three.
`internal/loomcli/doc.go` gains a sentence naming `lyx loom fmt` as the verb that rewrites named Go files to their `go/format` form, refusing the whole call on any path it cannot format.
`docs/overview.md` adds `fmt` to loom's verb list and one line describing `fmt <file>...` beside the `lint-comments` line.

**ImpactSummary:** Adds one lightweight loom verb and its spec rows; the only existing behavior touched is `resolveRunID` no longer reading `fmt`'s argument as a run-id.
