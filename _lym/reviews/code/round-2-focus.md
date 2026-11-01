---
round: 2
focus:
  - "Does every path that creates a board project or sets its slug in internal/boardengine (NewTask in task.go, and any patch that renames or moves an entry) refuse the slug equal to RootSubject (owners.go), with a message naming the reserved slug?"
  - "Does Render in internal/boardengine/render.go build the root project's file name from the RootSubject constant via projectFileName, with no repeated literal \"root\" (PATTERN-named-constants)?"
  - "Does a test pin each refusal, and does the claim list in internal/boardengine/doc.go state that project-root.md is the root project's file and not a project's?"
---

Focus covers the reserved-slug refusal across creation and patch paths, the named constant, and test and doc coverage.
