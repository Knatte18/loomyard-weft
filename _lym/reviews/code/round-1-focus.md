---
round: 1
focus:
  - "internal/boardengine/layer.go `ComputeLayers`: does an open draft task held by a run stay Running, and is the draft check placed so a draft task is never Ready, a letter layer or Independent, with open dependencies or isolated?"
  - "internal/boardengine/layer.go depth computation: does a task depending only on a draft task land in Layer A, and does cycle detection still follow draft tasks, given the draft task is now pre-assigned a layer?"
  - "internal/boardengine/layer.go `RenderOrder`: does the new bucket order place Draft after Independent and before done without shifting any other bucket or breaking the letter range A..Y?"
  - "internal/boardengine/render.go and the project member table: does a draft member's standing read `Draft`, and does the Tasks section write `### Draft (n)` last, omit it when empty, and keep draft tasks out of Ready's count?"
  - "internal/boardengine/doc.go against the diff: does the README part of the package doc state the Draft subsection, and does the Layer A meaning line match render.go?"
  - "Tests in layer_test.go and render_test.go: does each acceptance criterion of the decision record have coverage, and do the edited existing tests duplicate one another?"
---

Scope is six files under `internal/boardengine`.
The decision record's last entry bars touching the docs/research bench files and the link allowlist, so check the diff stays out of them.
