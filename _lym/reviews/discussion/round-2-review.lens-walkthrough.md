```yaml
lens: walkthrough
outcome: reported
```

The walkthrough lens ran one concrete case through each decision's mechanism, D1 to D8.

- **D1:** the `darn-write` key set to `scope: coverage`, called with no targets, and Darn's `tree_scope` call with explicit targets. `computeScope` gives explicit targets the `scopeTargets` form untouched, and the rest takes the `diff` path. Purpose met.
- **D2:** package B's test calls package A, built with coverpkg covering A and B. `fillMapTests` takes the block table from the first profile. `KeyBlocks` needs each covered package's own directory; that is plan detail. The marker survives an older binary's `WriteMap`, because `state` keeps undeclared keys. Purpose met for Go files. It is not met for embedded files below a package directory (finding 1).
- **D3:** a diff to `internal/configreg` with an importing package P. The changed set runs whole, and P runs its D3 union. Purpose met.
- **D4:** an edit inside `F` of A. The working-tree hash differs, `CodeFileAt(map.Commit)` gives the old text, and both sides are parsed. `F` is in both with a different body. B's tests whose blocks name `F` are selected. Purpose met on a checkout whose working-tree bytes equal the blobs. It is not met under line-ending conversion (finding 2).
- **D5:** an edit to `contracts/stencils/loom/loom-template-discussion.md`. impactset maps it to package `contracts/stencils`, which is changed, so its importers are narrowed. No Go file changed, so D4 selects nothing. The file is not "a non-Go file of a covered package's directory", because only files directly in the directory are recorded. No miss fires and nothing is selected. Purpose missed (finding 1).
- **D6:** P's selected tests run in a second invocation with `-run ^(TestA|TestB)$`. `recordPass` writes an entry with those args, and `newestWholeAncestor` skips any entry with args. When the gate later runs over the same tree, a whole empty-args package entry covers that invocation via `isGuardRunFilter`, which is correct. A selected entry never matches a request with empty args. Purpose met.
- **D7:** the labels follow from D3, D5 and D6. Purpose met.
- **D8:** a `ReadMap` error becomes a D5 miss, and no build starts. Purpose met.

Findings:

1. **BLOCKING (design): D2 and D5 miss files a covered package embeds from a subdirectory, so importers' tests are skipped.**
   - **Problem:** D2 records "every file its directory tracks" for a covered package. That list comes from `CodeDirAt`, which returns only the files directly in the directory (`internal/fabricengine/codereads.go:12-21`, the listing `testmap.go:427` uses). D5's non-Go miss covers only those files.
   - **Counterexample:** `contracts/stencils/stencils.go:16-21` embeds `landing/...` and `loom/...` files from subdirectories. impactset maps such a file to package `contracts/stencils` (`internal/impactset/derive.go:105-112`), so the package counts as changed. Its importers are then narrowed. The map holds no record of the file, no Go function changed, and no miss fires. The importers select no test, even when their tests render that embedded text. Only the landing gate catches it.
   - **Bound:** no stated bound covers this.
   - **Fix:** record and compare every tracked file under a covered package's directory, excluding nested package directories. Or make a changed file below a covered package's directory a miss.
2. **LOW (design): D4 and D5 compare blob hashes with working-tree bytes, so line-ending conversion defeats the narrowing.**
   - **Problem:** map hashes come from blobs at `Commit` (`internal/gatecli/testmap.go:436-441`, through `CodeFileAt`). D4 and D5 hash working-tree files.
   - **Case:** on a checkout with `core.autocrlf` converting Go files, every file reads as changed. The repo's `.gitattributes` (lines 1-6) shows such checkouts exist and pins LF only for listed files. Every importing package then misses (D5's own-`Files` rule) or selects every test whose blocks are in the converted files.
   - **Effect:** it is never unsafe, but on such a machine the decision's purpose is lost and nothing states that bound.
   - **Fix:** hash the working tree through git's clean filter, or state the bound.

Tool calls:
- grep for `//go:embed`, `.gitattributes` and `impactedSet`: to check whether embedded and non-Go files reach covered packages, and how importers join the set. The manifest held none of these.
- grep for non-template `//go:embed` lines: to confirm subdirectory embeds exist (`contracts/stencils/stencils.go`). The first grep's output was truncated.
