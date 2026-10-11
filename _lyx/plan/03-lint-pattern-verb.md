# Card 3 — lint-pattern-verb

**Create:**
- `plan:internal/patternlint#Lint` -> `func Lint(worktree, base string) (pattern.Finding, bool, error)`
- `plan:internal/patternlint#WayForward` -> `const WayForward = "cut the named bytes from PATTERN.md, moving detail into the entries' background files"`
- `plan:internal/patternlint#readOverview` -> `func readOverview(worktree, revision string) ([]byte, error)`
- `internal/patternlint/doc.go`
- `plan:internal/loomcli#loomCLI.lintPatternCmd` -> `func (c *loomCLI) lintPatternCmd() *cobra.Command`
- `plan:internal/loomcli#TestLintPatternCmd_OverARepository` -> `func TestLintPatternCmd_OverARepository(t *testing.T)`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/pattern#Headroom`
- `internal/commentlint/diff.go`
- `internal/loomcli/lintcomments.go`
- `internal/loomcli/lintcomments_integration_test.go`

**Intent:** Add the `lyx loom lint-pattern` verb of decision D3 and the small git-reading package both it and the round gate call, so the verb and the gate sites share one path onto `pattern.Headroom`, per `PATTERN-gate-self-check-parity`.
`internal/patternlint` is a new package shaped like `internal/commentlint`: `Lint` is told the worktree and the base and resolves no cwd.
With an empty base it compares HEAD's overview with the working tree's; with a base commit it compares that commit's with HEAD's, which are the two base modes `lint-comments` has.
`readOverview` reads `PATTERN.md` at a revision through `gitexec.Run`, the same git read path `commentlint` uses, never raw git: it first runs `git ls-tree <revision> -- PATTERN.md` and answers nil for an empty listing, so an overview absent at the revision counts as zero bytes rather than a failed `git show`, and otherwise returns the bytes of `git show <revision>:PATTERN.md`;
an empty revision reads the working tree file through `os.ReadFile`, answering nil when it does not exist.
`Lint` hands both contents to `pattern.Headroom` and returns its finding; a git or read error is returned, never a finding.
`WayForward` is the one statement of the fix the verb and the round gate both name.
`internal/patternlint/doc.go` carries the package doc: the two base modes, the absent-overview rule, the bound of decision D2 that it blocks only growth above the line, and that it writes nothing.
`lintPatternCmd`, in a new `internal/loomcli/lintpattern.go`, builds the cobra verb exactly as `lintCommentsCmd` does: `Use` `lint-pattern`, a one-line lower-case `Short`, a `Long` under the byte cap naming the two base modes and the example, the operator audience annotation, `Args: cobra.NoArgs`, the `--base` flag with the same meaning, and the `clihelp.ShouldAbort` check first.
On a finding it writes one failure envelope through `output.ErrFields` whose message names the way forward and whose `findings` key holds the finding's message as a one-element list, and sets exit 1; on a clean comparison it writes the ok envelope with the worktree and exits 0; a `Lint` error is the error envelope naming the worktree.
`Command` adds the verb beside `lintCommentsCmd`, and `verbUsesLightweightWiring` adds `lint-pattern` to the module-config-free set, with its doc comment naming the two lints, since the verb reads only git and files; `TestVerbUsesLightweightWiring` and `TestResolveRunID` each gain the matching row.
`TestLintPatternCmd_OverARepository`, in a new `internal/loomcli/lintpattern_integration_test.go` under the `integration` tag, walks one repository as the comment lint's test does: it commits a `PATTERN.md` sized just under `pattern.HeadroomBytes`, grows it in the working tree past the line and asserts exit 1, an envelope with `ok` false and a `findings` list holding one message naming the bytes to cut; shrinks it back and asserts a clean pass; commits a growth past the line and asserts that the working-tree form passes while `--base <first commit>` fails with one finding.
The envelope shape it asserts is the one `TestLintCommentsCmd_OverARepository` asserts, so the two verbs stay alike.
The verb gets its own test rather than a step in the comment lint's, because that test's fixture and name are the comment verb's; the `internal/loomcli` row of `cmd/lyx/testdata/test-budget.yaml` rises by one.
`docs/overview.md` gains `internal/patternlint` in the module table beside `internal/commentlint`, `lint-pattern` in the loom verb list, and a description of the verb after the `lint-comments` one.

**ImpactSummary:** One new leaf-adjacent package and one new loom verb; the only existing code changed is the verb registration and the lightweight-wiring set, both pinned by their tables.
