```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I re-ran the set searches for F1 and F3, the findings whose premise is a set of sites. F2 is a single file, so I left it unchecked.

**F1, stale carrier claims.** The search was over "pair teardown's commit", "landing's records commit" and "commit of uncommitted record". It found no remaining occurrence in code or docs, outside the review files. The fix covers the set.

**F3, consumer enumerations of `RunsRootRel`.**

- LOW: `internal/shedrun/doc.go:45` says "`RunsRootRel` names the anchor-relative run-records root, because fabricengine's Add drops everything under it from a freshly forked pair." This is the twin of the `paths.go:349` comment the fixer reworded. It still names one specific consumer and explains the symbol only through that consumer. This is the same pattern the review flagged ("enumerates its consumers"). The fix left it as it was. The sentence is true today and names a single consumer rather than a list, so I rate it LOW.
- The other hits describe their own subject and are not enumerations: `fabricengine/forkrecords.go:18`, `fabricengine/doc.go:681`, `pattern/PATTERN-shed-driver.md:96`, `docs/research/twill-plan-bench.md` (a measurement report), and `paths.go:357`, the `RunsRoot` doc.

**Unchecked:** F2, which targets only `_lym/landing/summary.md`.

**Tool calls used (2 of 6):**
1. `grep` for the F1 carrier phrases across `.go` and `.md` files: to confirm none survive, which my context could not show.
2. `grep` for `RunsRootRel` and the "stages it with the run directory" wording: to find other enumerations of its consumers and other stale `SettleRecords` staging claims.
