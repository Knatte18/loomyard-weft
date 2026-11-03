```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes for F1, F2 and F3 cover every site under each finding's key.

- F1 (remedy wording): no findings. A search of `*.go`, `*.md` and `*.yaml` found no remaining copy of the old text ("never joined to root", "spell it worktree-relative"). The only nearby text is the doc comment at `validate.go:1358`, "names the worktree-relative self glyph and the plain path root: does join". It still matches the new behavior.
- F2 (spec phrasing): no findings. Only `loom-plan-checks-spec.md:64` carried the "present" wording, and it now reads "present on disk or created by a card". No other spec states the condition.
- F3 (guard row): I left it unchecked. Its premise is a coverage gap, and the new row covers it by construction.

Low-severity gap, not a finding: `rootRelativeGlyphRemedy`'s doc comment says "when ... joined ... reports present", while the spec now says "on disk or created by a card". Both describe the `satisfied` closure, so nothing disagrees.

Tool calls:
1. One `grep` over the repo for the old remedy wording and the spec phrasing. My context held neither the search results nor files outside the three edited ones.
