```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- The old wording is gone. The grep for "line-break lint" and "wrapped breaks in" found no remaining hit outside run records. The `template.yaml` and `websterengine/doc.go` edits have no stale twin.
- `headingPattern` is still used at `lint.go:180`, the wrapped-break rule's paragraph check. It stays at level one, as the review asked, so the two heading patterns do not conflict. The new `anyLevelHeadingPattern` is defined and used only in `linelength.go`.
- The `lint.go:63` doc for `Lint` already names both rules.
- The `linelength.go` doc for `findNewLongLines` now says the skip happens once per occurrence in the base. That matches the counting map.

Tool calls:
1. `grep -rniE` for the old lint wording and `headingPattern`, to look for twins outside the files my context shows.
