---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/boardengine/render.go:477
    summary: An open project under a done project gets no README section, so its open members leave the README's view.
    fix: In projectSections, also emit sections for open projects whose parent chain passes through a done project, for example by treating an open project whose parent is done as a root-level one in childProjects(Task{}); keep the depth-first order, and add a golden row with an open project under a done project.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: consistency
    location: internal/boardengine/doc.go:172
    summary: The doc and the decision say root-level project sections follow the root's order key, but renderReadme passes the zero Task, so they sort by handle only.
    fix: Reword the doc.go README paragraph and the projectSections comment to say root-level projects run by handle, or pass the root's order into the render and apply it in childProjects.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: scope
    location: internal/boardengine/render_test.go:499
    summary: No project-section member carries a priority or a body, so the H/L cells and the linked slug are unpinned in project-section tables.
    fix: Give one member of the Ship project in the fourth golden row a high or low priority and a body, and extend the expected table rows.
    origin: lens:code-tests
---

### [MEDIUM] Open project under a done project is invisible in the README

**Location:** internal/boardengine/render.go:477
**Issue:** `childProjects` skips done projects and `projectSections` recurses only through it. Prune keeps a done project while an entry still names it, and no write rule found refuses marking a project done while it has open members. Such members appear in no README section, the failure the task exists to remove. Reachability was not proven.

### [LOW] Root order key claimed but not applied

**Location:** internal/boardengine/doc.go:172
**Issue:** `Render` has no root order, so `projectSections(Task{})` sorts root-level projects by handle only. The old Projects list behaved the same, so this is an over-claim in the new text.

### [LOW] Priority and slug cells unpinned in project-section tables

**Location:** internal/boardengine/render_test.go:499
**Issue:** Removing `priority: true` from `memberTable` would not fail the golden, since no member has a priority.

## Rejected

- Heading `\|` escaping from `cellText` in a `##` heading: CommonMark renders it as `|`, and Projects bullets did the same.
