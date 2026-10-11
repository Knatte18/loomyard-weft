# Card 3 — lint-pattern-verb

**Create:**
- `internal/patternlint#Lint`
- `internal/patternlint#WayForward`
- `internal/patternlint#readOverview`
- `internal/patternlint/doc.go`
- `internal/loomcli#loomCLI.lintPatternCmd`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `internal/loomcli#TestVerbUsesLightweightWiring`
- `internal/loomcli#TestResolveRunID`
- `internal/loomcli#TestLintCommentsCmd_OverARepository`
- `docs/overview.md`

**Uses:**
- `internal/pattern#Headroom`
- `internal/commentlint/diff.go`
- `internal/loomcli/lintcomments.go`

**Intent:** Add the `lyx loom lint-pattern` verb of decision D3 and the small git-reading package both it and the round gate call, so the verb and the gate sites share one path onto `pattern.Headroom`, per `PATTERN-gate-self-check-parity`.
`internal/patternlint` is a new package shaped like `internal/commentlint`: `Lint` is told the worktree and the base and resolves no cwd.
With an empty base it compares HEAD's overview with the working tree's; with a base commit it compares that commit's with HEAD's, which are the two base modes `lint-comments` has.
`readOverview` reads `PATTERN.md` at a revision through `gitexec.Run`, the same git read path `commentlint` uses, never raw git: it first runs `git ls-tree <revision> -- PATTERN.md` and answers nil for an empty listing, so an overview absent at the revision counts as zero bytes rather than a failed `git show`, and otherwise returns the bytes of `git show <revision>:PATTERN.md`;
an empty revision reads the working tree file through `os.ReadFile`, answering nil when it does not exist.
`readOverview` turns every CRLF into LF in what it returns, on both paths, so a checkout under `core.autocrlf` compares its working-tree file with the LF blob byte for byte and an untouched overview never counts as growth.
`Lint` hands both contents to `pattern.Headroom` and returns its finding; a git or read error is returned, never a finding.
`WayForward` is the one statement of the fix the verb and the round gate both name.
`internal/patternlint/doc.go` carries the package doc: the two base modes, the absent-overview rule, the bound of decision D2 that it blocks only growth above the line, and that it writes nothing.
`lintPatternCmd`, in a new `internal/loomcli/lintpattern.go`, builds the cobra verb exactly as `lintCommentsCmd` does: `Use` `lint-pattern`, a one-line lower-case `Short`, a `Long` under the byte cap naming the two base modes and the example, the operator audience annotation, `Args: cobra.NoArgs`, the `--base` flag with the same meaning, and the `clihelp.ShouldAbort` check first.
On a finding it writes one failure envelope through `output.ErrFields` whose message names the way forward and whose `findings` key holds the finding's message as a one-element list, and sets exit 1; on a clean comparison it writes the ok envelope with the worktree and exits 0; a `Lint` error is the error envelope naming the worktree.
`Command` adds the verb beside `lintCommentsCmd`, and `verbUsesLightweightWiring` adds `lint-pattern` to the module-config-free set, with its doc comment naming the two lints, since the verb reads only git and files; `TestVerbUsesLightweightWiring` and `TestResolveRunID` each gain the matching row.
The verb's tests extend `TestLintCommentsCmd_OverARepository`, the integration scenario that already covers `lint-comments`'s envelope, per the decision record's `PATTERN-test-economy` constraint, so no new top-level test is added and no budget row rises.
The scenario's repository also commits a `PATTERN.md` sized just under `pattern.HeadroomBytes` in its first commit, and its `lint` helper takes the verb to run, so each step asserts both verbs over the same tree.
Beside the wrapped comment, the first step grows `PATTERN.md` in the working tree past the line and asserts `lint-pattern` exits 1 with `ok` false and a `findings` list holding one message naming the bytes to cut;
the clean-tree step restores the committed overview and asserts both verbs pass, then writes the committed overview back with CRLF line endings and asserts `lint-pattern` still passes;
the committed-range step commits the growth beside the wrapped comment and asserts that `lint-pattern`'s working-tree form passes while `--base <first commit>` fails with one finding.
The test keeps its name, and only its doc comment and the file header of `lintcomments_integration_test.go` change, to name both verbs.
`docs/overview.md` gains `internal/patternlint` in the module table beside `internal/commentlint`, `lint-pattern` in the loom verb list, and a description of the verb after the `lint-comments` one.

**ImpactSummary:** One new leaf-adjacent package and one new loom verb; the only existing code changed is the verb registration and the lightweight-wiring set, both pinned by their tables, and the comment lint's scenario test, which grows to drive both verbs.
