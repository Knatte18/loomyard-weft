---
round: 1
focus:
  - "Leftovers of the old spelling: `git grep -i lyx` over the diff's tree, excluding `go.sum`, `docs/research/` and the `LyX` editor spelling; settle whether any tracked file or path still holds `lyx`, `Lyx` or `LYX`."
  - "Dobby geometry (`internal/dobbyengine/geometry.go` and its callers): settle that `MarkerDirName` and `MarkerDir` are gone, `LogPath` is `.lym/beam/dobby.log`, `TidyDir` is `.lym/beam/tidy/`, and tidy reports are named by UTC timestamp alone; check the board-tidy writer and any test or doc still naming `markers`."
  - "Mechanical case-mapping hazards: identifiers, package names, string literals, regexes, gate keys, hash or fingerprint tables and sorted lists where renaming changed ordering, width or meaning; settle whether any rename changed behavior beyond the name."
  - "Out-of-scope reach: settle that the diff adds no migration code, does not touch the weft side, `go.sum`, `docs/research/`, or the board entry, and that the `LyX` editor guard in the deploy test keeps its original spelling."
  - "Binary and deploy: `cmd/lym` exists as the build target, `binaryName()` in `tools/deploy`, `update-plugins.sh` and `.cmd`, `deploy-dev` and the `ly` plugin stencils and skills all invoke `lym`; settle that no script or launcher still calls the old command."
  - "Repository rules landing with the diff: PATTERN.md and `pattern/` entries naming renamed packages or variables are consistent with the code, and the module docs (`doc.go`, `docs/overview.md`) describe the renamed behavior including the dobby paths."
  - "Reflow commits: settle that the `godocreflow` comment changes alter line breaks only and no words, and that each commit message in the range is accurate."
---

Seed focus for the rename of `lyx` to `lym`; the diff is largely script output, so the review targets what a mechanical rewrite gets wrong and the hand-written dobby change.
