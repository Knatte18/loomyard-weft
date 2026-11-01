```yaml
lens: fix-consistency
outcome: reported
```

no findings

- doc.go:75 now agrees with its sibling at doc.go:193 ("a Notes row adds the Prio column after it"). No other doc.go line still ties Prio to project sections.
- docs/overview.md:365 says priority "orders it first or last within its subsection". doc.go:75 says "no view orders by it". The two disagree, but the disagreement predates the fix and the fix didn't touch overview.md:365. Out of scope (not a finding).
- No other `heading :=` or `heading =` declaration remains in `internal/boardengine/*.go`, so F2 has no unrenamed twin.

Calls made:
1. `grep -rniE "prio"` over `docs/overview.md` and `internal/boardengine/doc.go`: to find sibling Prio sentences, which my context lacked.
2. `grep -nE "\bheading\b *(:=|=)"` over `internal/boardengine/*.go`: to find other shadowing declarations or leftover assignments, which my context lacked.
