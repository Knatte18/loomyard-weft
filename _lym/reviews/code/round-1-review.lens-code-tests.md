```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour of the diff, the test row that pins it (lens code-tests).

**Per behaviour: pinning test, or none**

1. One README section per open project, nested ones included, depth-first, in the parent's order key then by handle: pinned by `TestRenderReadmeGolden`, the fourth row. The fixture nests `later` and `inner` under `ship`, with `ship`'s `Order: ["later"]`. The golden shows Ship, Later, Inner, Tidy. Breaking the depth-first recursion or the order would fail the row.
2. Heading is the linked handle and title, with the count of open members: pinned by the same row (`## [P1 Ship it](project-ship.md) (6)`).
3. Members are open only: pinned. The done task `shipped` (T4) is a `ship` member and is absent from the table, and the count is 6.
4. Members of every kind, in RenderOrder's order: pinned. The `ship` table holds tasks, projects and a note in that order. The running T2 sorts first, and the nested projects and note E1 follow.
5. Done projects get no section: pinned. P3 `old` has no section.
6. Brief shown when present and omitted when empty: pinned. Present for Ship, Inner and Tidy; absent for Later and the Lone beam.
7. `_None._` for an empty project: pinned by Later, Inner and the Lone beam row.
8. The old Projects bullet list is gone from the README: pinned, since the full-text golden no longer contains it.
9. Prime heading, meaning line and demoted levels, with the sections below the project sections: pinned. Every README row (first, second, third, fourth, fifth) carries `## Prime`, `### Tasks` and `#### Ready`. The fourth row shows Prime after the project sections.
10. Project pages keep their heading levels: pinned by the project-file goldens, which are unchanged apart from the Prio and grouping edits.
11. Prio header on Notes tables: pinned by every Notes golden.
12. Prio header on project-section tables: pinned by the fourth row.
13. The `grouping` line is dropped on a non-beam page: pinned by the inner and tidy project-file goldens.
14. A beam page keeps `beam` and its owner: pinned by the existing `ship` page golden and the fifth row. The diff leaves those assertions untouched.
15. `table` and `entrySections` extraction leaves rows and trailing blank lines unchanged: pinned by every existing golden, including the Done and Running tables.

**Findings**

- **LOW, scope, `internal/boardengine/render_test.go`, fourth row (~lines 480-500 of the fixture, with the project-section golden at ~499-545).** The project-section table has `priority: true`, but no fixture member under a project carries a high or low priority, so the H and L cells never render there. No fixture member has a body either, so the slug-link-to-design-doc cell is also unexercised in that table. Dropping `priority: true` from `memberTable` would keep the `Prio` header and blank cells, so the golden would still pass. The behaviour claimed in `doc.go` and the test comment ("H for high, L for low", "a Prio table") has no row. Fix: give one `ship` member a high or low priority in the fixture, and ideally a body, and extend the golden.
- No other behaviour is left without a pinning row.

**Tool calls**
1. `grep` of `render_test.go` around the "Ship it" fixture, to see the fixture's done members, priorities and projects, which the diff's context lines did not show.
