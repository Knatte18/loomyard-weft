```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the F1/F2/F3 change set (commit e9032bde).

Findings:

1. **LOW**: the doc.go README paragraph still claims "so every open entry shows in the README".
   - Changed text: the sentence the fix extended at `internal/boardengine/doc.go:171-173`. It now adds "An open project under a done project counts as under the root, since a done project has no section."
   - The fix only reroutes open projects. A done project still gets no section, and `projectSections` builds sections only from `childProjects`, which returns open projects. An open task or note whose `Project` is a done project still appears in no README section, because Prime holds only entries with an empty project.
   - The doc's "every open entry" claim therefore still overreaches for that case. It is the twin of the F1 premise, which was "open members leave the README's view", and the fix left it as it was.
   - Either say in the doc that members of a done project show only in that project's file, or give the README a section for them. Whether Prune or a write rule makes the case reachable was not checked (the review marks F1's reachability "not proven").

2. No other disagreement found. The doc.go reword (root by handle, nested in the parent's order key) matches `childProjects`, which sorts by `project.Order` and then by handle. The root's `Order` is nil, so root-level sections run by handle. The `childProjects` and `projectSections` comments agree with the code. The new test row and its `projectFiles` entry for `project-orphan.md` agree with the render order of the new `Orphan` section, which sits after Tidy by handle P6.

Further tool calls: none; my context already held every changed file and the diff.
