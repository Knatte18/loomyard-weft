no findings

`isTopBoxRule` has one call site; every `wantHeldNotice` row sets `wantHeldReason`; the notice text appears at one site.
`editloop.go` and `dialog.go` also build `Shuttle notice:` lines, outside F2's named site and untouched by the fix.

Tool calls:
1. grep Go sources for `cutLabelRuleEnd`, `isTopBoxRule` and `Shuttle notice: `; the context did not list call sites.
2. grep Go and markdown for `never took its start prompt`; to find stale assertions or docs.
