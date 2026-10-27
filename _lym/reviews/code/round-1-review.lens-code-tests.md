```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. Per behaviour, the test that pins it.

| Behaviour | Pinning test |
|---|---|
| Open draft task with no dependency is `Draft`, never Ready | `layer_test.go:69-79`, row "a draft task is Draft…", slug `free`. Also the README golden at `render_test.go:298` and `render_test.go:507`. |
| Draft task with an open dependency is `Draft` | Same row, slug `waiting`. |
| Isolated draft task is `Draft`, not Z | Same row, slug `alone`. |
| Task depending only on a draft task lands in Layer A | Same row, slug `after` = `A`. |
| Run-held task stays `Running` whatever its status | Only an indirect pin. The existing row at `layer_test.go:~66` has `held` = Running but I could not confirm it carries a draft status, so the case "draft status and run-held" is likely **none**. In the draft check order, a draft status plus a run lock is unreachable per `draft.go:5` ("a draft entry holds no run lock"). The check sits after `RunHolds` anyway, so a regression is low-risk. |
| Released task with no open dependency stays Ready | Existing rows ("task": "Ready" at `layer_test.go:67`) and the golden's `T1`/`first`. |
| `RenderOrder` places Draft after Z and before done | `layer_test.go:168-175`, an added `d1` in the existing ordering test. It pins Z < Draft < done. |
| README `### Draft (n)` renders last, with the meaning line and the draft title cell | `render_test.go:298-304` (golden 1) and `render_test.go:507-513` (golden 2). |
| `### Draft` omitted when empty | Pinned only by the other goldens that hold no draft task, if any. Golden 1 at `render_test.go:~406-409` ("Layer A" block with no Draft) probably covers it. No dedicated row. |
| Ready counts no draft task | `render_test.go:276` (`### Ready (0)` + `_None._`) and `render_test.go:491` (`### Ready (1)` with `T5` removed). |
| Project member table standing reads `Draft` | `render_test.go:477` (`T5` … `Draft`). |
| Layer A meaning line text | `render_test.go:284`, `409`, `501`. |
| Cycle detection still follows draft tasks | **None.** The diff pre-assigns the draft layer but leaves the DFS unchanged, and no test cycles through a draft task. A regression here would not be caught. |

Findings:

1. **LOW / scope** — `layer_test.go:69-79`. The cycle-detection path through a draft task is uncovered. The focus directive asks about it, and `ComputeLayers`' doc says cycle detection "still follows running tasks" but now also covers draft ones. A cycle row has no test pairing it with draft. Suggested fix: add a row, or a short error-path assertion, where two tasks, one of them draft, depend on each other, and expect the `cycle detected` error. This is a coverage gap in behaviour the diff changed indirectly, not a redundancy.

2. **NIT / design** — `layer_test.go:69-79`. Test economy: one combined row asserts four behaviours (free, dependency, isolated, depends-on-draft). It is not redundant, but one failure message hides which sub-case broke. Optional, not blocking, and the repo's table style accepts it.

Test economy: no duplicated coverage found. The edited goldens in `render_test.go` change existing assertions instead of adding tests, and the `RenderOrder` row extends an existing test. There are no stencil-prose assertions and no per-helper tests for `isDraft`. The two golden edits repeat the Draft subsection check, but they cover different shapes (the plain Tasks section and the project member table), so I do not count that as duplication.

Tool calls (1 of 6):
1. `grep` for draft/`StatusDraft` in `layer_test.go`, `render_test.go` and the other `*_test.go` files, to see whether an existing test already covers draft layers or a draft cycle, since the manifest held only the diff hunks.
