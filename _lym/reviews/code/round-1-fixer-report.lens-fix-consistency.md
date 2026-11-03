```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change in `docs/overview.md:367` against its sibling texts.

No findings.

The new clause says a `Sketch` link to `decision-<slug>.md` sits under the slug of a discussed entry. That agrees with `internal/boardengine/doc.go:206` and `:294`, which describe the same link in the same cell. A repo-wide grep (`.md` and `.go`, `_lym` excluded) for the old phrasing "links each slug to its design doc" found only the line the fixer edited. No twin sentence was left unchanged.

Tool calls:
1. `grep -rn` for the old phrasing across `.md` and `.go` files: to look for twin descriptions of the Slug cell elsewhere, which my context did not show.
