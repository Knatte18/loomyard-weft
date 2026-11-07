```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Fix-affected-set lens. The two findings with a set premise were F4 (sites describing the lint as breaks-only) and F5 (sites using the old lint name). I ran one search for each, plus one for the heading sites.

- **F4 and F5:** no stale description remains outside the changed lines. `lint.go:63` is the `Lint` doc, which names both rules in its next line. The `lintcomments.go:29` help, `template.yaml:17` and `docs/overview.md:1145` all name both rules.
- **F1:** `anyLevelHeadingPattern` is used only for the length exemption. `headingPattern` is still used only by the wrapped-break check, as the review asked. `doc.go:55` states the exemption without a level limit, so it matches the fix.
- **F2 and F3:** neither has a set premise, so I left both unchecked.

Calls made:
1. `grep` for the break-only wording and old lint name across `.go`, `.yaml` and `.md` files. This covered F4 and F5, and my context had no such search.
2. `grep` for the heading-pattern uses in `internal/commentlint`. This covered F1's sites, and my context had no such search.
