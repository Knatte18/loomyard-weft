```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- **`childProjects`, root call, open project under a done project.** Take `orphan` (`Project: "old"`, `old` done). `bySlug["old"]` is found and done, so `under` is true. The entry is open and a project, so it is returned. The README test pins this with the new `## [P6 Orphan]` section. Without the fix that section is absent, so the test would fail.
- **Root call, ordinary entries.** Root-level entries pass through the first `under` clause. `inner` (`Project: "ship"`, `ship` open) gets `under` false. A root-level entry with an empty `Project` looks up `bySlug[""]`, which is not found, and the first clause already holds.
- **Non-root call.** With `project.Slug != ""` the new clause is false, so behavior is unchanged for `projectsSection` and for the recursion in `projectSections`.
- **No double emission in the README.** An open project reaches the README through exactly one path. It is root-level if its parent is empty or done, otherwise it is nested under its open parent.
- **Termination.** Recursion starts from a child with a non-empty slug, so the root clause cannot re-enter.
- **Root ordering.** `project.Order` is nil, so every position equals `len(nil)` and the sort falls to `compareHandles`. The expected P1, P2, P6 order matches, and the doc and comment rewording agrees with that.
- **F3 rows.** Dropping `priority: true` from `memberTable` would break the `H` and `L` cells. Removing `Body` support from `slugCell` would break the `[`second`](design-second.md)` link. Both assertions fail with the defect in place.
- **Gate.** `lym gate test ./internal/boardengine` passed (516 passed).

Minor, not a finding: the open-under-open-under-done case and the new Projects bullet in `project-old.md` have no golden assertion. The review did not ask for either.

Tool calls made: none; my context already held every file in its changed form.
