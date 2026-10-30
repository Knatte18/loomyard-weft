```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the finding-set searches for F1 and F2 (F3 has no set premise) and check the fix covers every member.

**Findings**

1. LOW. `internal/boardengine/render_test.go:274` and `docs/overview.md:362` are twins of the changed text.
   - The test doc comment says README.md holds "one section per open project, nested ones included, depth-first in each parent's order and then by handle".
     That is imprecise now that root-level projects run by handle only.
     It also omits the open-project-under-a-done-project case that the new `orphan` row pins.
   - `docs/overview.md:362` says "so no entry leaves its view". It still holds, and it is the claim F1 restores.
   - `render.go:4` and `render.go:335` are generic ("one section per open project on the board") and are consistent.
   - Evidence: the F2 search for "order key" and "root's order" found no remaining claim that root-level projects follow a root order key. The only stale-reading wording is the `render_test.go:274` comment.

2. F1 coverage is complete for the README. The one site that enumerates a README's open projects, `childProjects`, now covers the root. `projectSections` recurses through it, so open projects below an open project under a done project are reached from that open project. The fix does not change `projectsSection` for a done project's own file. A done project's file still lists its open children, which is consistent with its "every project renders" contract.

3. Unchecked: the F3 premise is a single test row, not a set, so no search was run for it.

Calls made:
- grep for "root's order|order key|one section per open project" across `.go` and `.md` files: the F2 premise is a set of sites claiming a root order key, and my context held only `doc.go` and `render.go`.
- grep for `b.bySlug`, `isDone(parent` and `parentOnBoard` in `render.go`, plus `git show --stat HEAD`: to check the F1 change's other users of `bySlug` and the commit's file set.
