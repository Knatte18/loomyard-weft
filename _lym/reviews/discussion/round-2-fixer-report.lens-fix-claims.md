```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked each claim the fix added or rewrote against the code at HEAD.

**LOW**: the measurement under-reports misses.
- Changed text: Constraints, "Of those, 94 hold a D5 non-body miss, and 102 would if added consts, types and vars counted as misses too."
- Summary: 94 is a lower bound under the rewritten D5, but the bullet states it as an exact count.
- Evidence: the scratch classifier `.scratch/nonbody/main.go` has three gaps against the rewritten D5.
  - It never looks at `//go:` directive lines, which the new D5 bullet makes misses.
  - Its `classify` passes every added import under `modulePath` as no miss. D5 exempts only a package "the map's linked set holds", and a newly imported in-module package may not have been linked when the map was built.
  - Its round-2 count also counts a signature change with an identical body as a miss, so 102 is not only "added consts, types and vars".
- Way forward: say "at least 94", or name the classifier's approximations.

**LOW**: "discards" goes further than what testmap does.
- Changed text: D2, "When one is dirty, it discards that package's map, as it does for files changed at HEAD."
- Summary: for files changed at HEAD, testmap does not discard a map. It skips the package, writes no new map and keeps the old one.
- Evidence: `internal/gatecli/testmappackage.go:95-97` runs `r.skip(pkg, "files changed under its tests: ...")` and then `return nil` before anything is written. "Discards that package's map" can be read as deleting the existing map, which is not what the code does.

**LOW**: the path-to-package rule is restated too broadly.
- Changed text: D2, "every tracked file impactset's path-to-package rule maps to it: every file under its directory outside a nested package directory, `go:embed` targets included."
- Summary: impactset's rule maps a `.go` file only to its own directory's package. A `.go` file under `testdata/` maps to no package, but the restatement includes it.
- Evidence: `internal/impactset/derive.go:101-103` returns found=false for a `.go` file whose directory is not a package. Only non-Go files walk up to the nearest enclosing package (lines 105-113).

These claims hold:
- `CodeDirAt` returns only the files directly in a directory (`internal/fabricengine/codereads.go:12-21`).
- The CRLF bound holds: own `Files` holds blob hashes of Go files only (`testmap.go:426-445`), and `.gitattributes` pins LF for no `.go` file.
- The per-package counts match the classifier's output.

Further tool calls:
- grep of `internal/gatecli/testmap.go` for `changedFiles`, the discard and `r.dirty`, to find the after-tests check D2's new sentence compares itself with. My context lacked where it lives.
- grep of `internal/gatecli/testmap*.go` to locate that check. It is not in `testmap.go`.
- Read of `internal/gatecli/testmappackage.go:190-234`, for the `changedFiles` body, which my context lacked.
- grep of `internal/gatecli/testmappackage.go` for `changedFiles(` call sites, to see what testmap does when files changed (it skips the package). My context lacked the call site.
