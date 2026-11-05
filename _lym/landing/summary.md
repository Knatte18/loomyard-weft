# Distill PATTERN.md into cluster entries with detail in background files

`PATTERN.md`'s overview sat at its headroom line (`MaxOverviewBytes` and `HeadroomBytes` in `internal/pattern/check.go`).
Every run that added or widened an entry stopped at Plan-Write until trims of other entries were recorded.

This change groups entries that guard the same subject into one cluster entry with one short rule line.
The overview now sits below 60% of `MaxOverviewBytes`, so several runs can add entries before the next trim.
The section headings are unchanged, and each cluster lives under the section its subject belongs to.
The cap and headroom values are unchanged.

## What moves

- Text that leaves the overview moves verbatim into the cluster's background file under `pattern/`.
- An entry folded into a cluster brings its own background file's text along, and that file is deleted.
- Some entries had no background file; their full text now lives in the cluster's file.
- Entries that were not folded keep their IDs, with lines shortened where detail moved to their background file.
- Every citation of a folded entry's ID in code, tests, stencils, specs and docs is rewritten to the cluster's ID.
  Folded entries whose file was renamed (for example `PATTERN-fabric-git` to `PATTERN-fabric-boundary`) are covered by the same rewrite.
- `internal/pattern/doc.go` gains one rule: on a cluster entry, `(test)` means at least one member rule is test-enforced, and the background file marks each review-only member rule.

## Design decisions

- Clusters are named for the subject, not for any one member, so a new rule on that subject has an obvious home.
- A cluster's `(test)` marker would otherwise overstate what is enforced, hence the `doc.go` rule above.
  The background files carry the per-rule marking.
- No rule is dropped; the change is a regrouping and a move of text.

## To check by hand

- Run `grep -rn` for each folded entry's old ID.
  It should find nothing outside `docs/research` and history.
- Spot-check a few cluster background files against the old overview lines and old background files, to confirm the text moved verbatim and no rule was lost.
- Confirm each cluster's `(test)` marker is right: at least one member rule is enforced by a test, and review-only members are marked in the background file.
- Run `internal/pattern`'s checks.
  They were not run as part of writing this description.
