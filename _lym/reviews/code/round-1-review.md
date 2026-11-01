---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/boardengine/render.go:105
    origin: lens:code-correctness
    summary: A board project with slug root renders to project-root.md and overwrites the root project's file, with no error and no reserved-slug check
    fix: In internal/boardengine/task.go NewTask, refuse slug RootSubject (owners.go) with a message naming the reserved slug, next to the validateSlugLength call, and build rootProjectFile in render.go as projectFileName(RootSubject) instead of the literal "root". Add a refusal row to the NewTask slug-validation test. Update the claim list at internal/boardengine/doc.go:338 to say project-root.md is the root project's file and not a project's.
---

### [MEDIUM] Project slug `root` collides with the root project's file

**Location:** `internal/boardengine/render.go:105`, `render.go:107-111`, `render.go:163`.
**Issue:** `Render` stores the root's file under `projectFileName("root")` first, then the loop over `ordered` writes `projectFileName(twl.Slug)` for every project.
A project with slug `root` replaces the root's file, and the root section's heading link and that project's own link then point at the same file.
`Render` returns no error and nothing in README.md shows it.
`NewTask` checks only that the slug is non-empty and within `MaxSlugLength`, and a search of `boardengine` and `boardcli` found no reserved slug.
The slug is plausible, since `RootSubject` is `"root"`.
The literal `"root"` in `rootProjectFile` is also a repeated name that `PATTERN-named-constants` asks to be one constant.
Three lenses raised the collision; the doc.go claim-list line and the missing test row they listed separately are covered by this fix.

## Rejected

- code-affected-set F2 (LOW, `doc.go:338` claim list): contingent on F1's fix and folded into F1's `fix`.
- code-tests NIT (no `project-root.md` row for a root without sub-projects): the body path is the same `entrySections` call the README rows pin, so a row would duplicate coverage.
- code-tests LOW (no test for the slug collision): folded into F1, whose fix adds the row.
- Focus items on README heading levels, the root member count, the Done table, `upLink`, the facts line and the removed names: the lens runs and my own reading found them correct and fully landed in `render.go`, `doc.go` and `docs/overview.md`; no consumer of the removed names remains.
