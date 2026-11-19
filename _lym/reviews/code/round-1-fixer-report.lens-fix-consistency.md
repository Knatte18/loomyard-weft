```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 change set for twins left stale (fix-consistency lens).

No findings.

- F2: `Row.RoutedTo`'s first doc line (`haltnote.go:50`) still says "handed the run on to". Its second line and the `routedPrefix` comment now say "routed … by a routed Stuck or a reroute". Both wordings cover reroute rows, so they don't disagree. The other "routine Stuck" mentions are in `shedadapters`, where they are correct, because those adapters do return a routine Stuck.
- F1: the only other text that matches is the stencil rule at `contracts/stencils/burler/burler-review-format.md:50`. The review says that rule and its two remedies were kept, so it needs no change. The new regex's "not a quoted fragment followed by trailing prose" assertion matches the hint text in `verdict.go`.

Tool calls (2 of 6):
- `grep` for "routine Stuck", "Stuck hand-off" and "handed the run on" in `internal`: to find twin comments on the routed field. My context held only the two files the fixer edited.
- `grep` for "quoted fragment", `malformedQuotedSummary` and `quotingHint` repo-wide: to find other places that describe or depend on the hint. My context lacked the stencil file and any other users.
