```yaml
lens: fix-affected-set
outcome: reported
```

No findings. I did not run the two touched recursions (`met`, `standsFor`) on a loop; this is from reading the code and the search results.

**F1 (membership recursion).** I searched every `.members[`, `waitsOn(`, `standsFor`, `.met(` and `metBelow` site outside tests:
- `dependency.go` `met` and `standsFor` now use a path set (`metBelow`, `standsForBelow`). Together with `overlap`, they are the only recursive walks over `members`. `overlap` calls `standsFor`, so it is covered.
- `layer.go:73-74` (`detectCycleDFS`) calls `met`, which now terminates, and its own colour marking reports a loop as a cycle.
- `layer.go:108-112` (`getDepth`) follows the same unmet-member edges that `detectCycleDFS` walks first. A loop returns an error before `getDepth` runs, so leaving `layer.go` unchanged is sound. This departs from the review's "keep getDepth safe" wording, so the fixer-report should say it.
- `store.go:397-405` hands `waitsOn` and `members` to `findCycle`, which has its own `onPath` and `finished` state, so it terminates on a loop.
- `siblings.go:63,79` and `render.go:596` only call `met` and `overlap`, so they are covered through those.
- No `standsFor` loop case is reachable from the new tests, because it is unexported and a write refuses a loop. That matches the review's requested scope (`Pending` and `ComputeLayers`).

**F4 (After-column wording).** Searching for "open entries" wording finds one remaining site, `render.go:128`: "An entry waits only on the open entries it names under After…". It is rendered output text that existing README goldens pin. The review named only the line-13 header, and that wording is arguably still accurate, so I did not raise it as a defect.

**F2 and F3.** Neither has a set premise. Each adds one row, so I did not search for more members.

Calls made:
1. `grep` of membership-walking sites in `internal`: to enumerate every recursion site under the F1 key.
2. `grep` of "open entries" wording: to enumerate the F4 sites.
3. `Read` of `store.go:385-455`: my context lacked `findCycle`, and I needed to confirm it guards against loops.
