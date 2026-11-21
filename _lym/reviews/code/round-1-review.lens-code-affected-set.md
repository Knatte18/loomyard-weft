```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the affected set, and whether the diff covers it**

1. **`baseOverlay` signature** (drops `unrestorable`, takes `base redundancyTree`). The set is its caller at `redfirst.go:199` and its test at `redfirst_test.go:130`. Both are updated. `holdsGoFileAtOrAbove` is removed and nothing still references it (grep over `*.go`). Covered.
2. **The unjudged exit (5) now means only a scoper that is not go.** The set:
   - help text in `redfirst.go:72` and doc comment at `:104`: covered.
   - `gatecli/doc.go:855` and `:890`: covered.
   - `gateslot/doc.go:326` and `gateslot/redfirst.go:14-16`: covered.
   - the failure text in `loomshed/redfirstgate.go:101-103`: covered.
   - the `loomshed/redfirstgate.go:33` doc comment says "a check that cannot judge the diff". That wording is generic, so it is still correct.
   - `shedrecipe/doc.go:209` says "a check that cannot judge the tests". Also generic and still correct.
   - the stencil `darn-template-write.md:111`: covered.
   - `docs/overview.md:993-994`: covered.
   - `loomshed/doc.go:178` says "for a hub or a diff the check cannot judge". **Not covered**: no diff can be unjudged any more.
   - the test `redfirstgate_test.go` row for exit 5: covered.
3. **The overlay now also restores embedded non-Go files.** The set:
   - `redfirst.go:3` file header, help at `:65-66`, and `gatecli/doc.go:869-874`, `:899`: covered.
   - `docs/overview.md:993`: covered.
   - the `run` doc comment at `redfirst.go:189` still says "lays the base's production files over the working tree". **Partly stale**: the file header was updated to name the embedded files, but this comment was not.
4. **The stencil clause dropped** ("or when the diff changes a non-Go file of a package"). The Goal asks for this. No other rule in the stencil paragraph is weakened. The "or the check cannot judge it" route still stands for the scoper case. Covered.
5. **Repository rules.** `PATTERN-code-conventions`: the embed prefix is one named constant, `goEmbedDirective`, and no language, engine or branch is hard-coded beyond the existing go scoper. `PATTERN-gate-slot.md` and `loom-discussion-spec.md` mention red-first but not the non-Go cause. No PATTERN changes were declared, so the "Declared PATTERN changes landed" check has nothing to verify.
6. **Commit messages.** `4abc6f41` carries a `Root-Cause:` trailer naming the overlay's classification defect, which the diff removes. `2bf05f59` only rewraps comments. `fe4aa057` changes only the loomshed test's expected text, which its subject describes. No `Pattern-Departure:` or `Loosened-Check:` trailers. The `fe4aa057` row tightens an assertion rather than loosening it.

**Findings**

- **LOW, consistency.** `internal/loomshed/doc.go:178`: the package doc still says exit `gateslot.RedFirstUnjudgedExit` happens "for a hub or a diff the check cannot judge". After this diff only a hub whose scoper is not go produces it, and `gateslot/redfirst.go:15` and `loomshed/redfirstgate.go:102` now say so. Evidence: `internal/loomshed/doc.go:178`.
- **NIT, consistency.** `internal/gatecli/redfirst.go:189`: the `redFirstCall.run` doc comment says it "lays the base's production files over the working tree". It omits the embedded non-Go files that the diff's `baseOverlay` now also restores, while the file header at `redfirst.go:3` was updated to name them. Evidence: `internal/gatecli/redfirst.go:3`, `:189`, `:355-356`.

**Tool calls**
1. Grep for the old phrasing and `RedFirstUnjudgedExit` across `*.go`, `*.md` and `*.yaml`: the manifest held no repo-wide list of its readers.
2. Grep for the `.md` files that mention red-first, and for removed symbols: needed the docs set and a check for leftover references.
3. Read `loomshed/doc.go:172-182`, `redfirstgate.go:28-36`, `shedrecipe/doc.go:203-212`, the PATTERN and spec hits, and `gatecli/doc.go:848-866`: these readers of the changed meaning were outside the manifest.
4. `git show --stat` of `2bf05f59` and `fe4aa057`: needed each commit's file set to judge its message.
