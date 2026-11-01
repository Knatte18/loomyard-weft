```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Behaviours the diff adds or changes, and the test that pins each:

| Behaviour | Pinning test | Result |
|---|---|---|
| Registry holds `loom-plan-checks-spec` and `loom-plan-csharp-spec`, in order | `TestRegistry_NamesAndDefaults`, `specs_test.go:24-33`; breaks if either entry or its position is wrong | pinned |
| The two new specs embed non-empty bytes | Same test, the loop at `specs_test.go:35-44` | pinned |
| The two new specs deploy under `loom/` | `TestRegistry_RelPathPlacesBothUnderLoomFamily`, new rows at `specs_test.go:60-61` | pinned |
| The `LoomPlanChecksSpec` and `LoomPlanCSharpSpec` embed directives and vars | The non-empty-bytes loop above; an embed of the wrong file would still be non-empty, but that is not testable behaviour | pinned |
| Stencil edits (`loom-block-plan-*`, `loom-rubric-plan-review`, `webster-template-plan-fix`) | None; static prose, exempt | exempt |
| Spec content: `loom-plan-spec.md`, the new `loom-plan-checks-spec.md`, `loom-plan-csharp-spec.md`, and the part scope and join text added to `loom-parts-spec.md` | None; static prose, exempt | exempt |
| Comment-only edits in `internal/planglyph/doc.go`, `internal/planparser/validate.go` and `internal/planparser/validate_test.go` | None; the test assertions are unchanged | no behaviour |
| `internal/webstercli/validate.go` `Long` help string now names `loom-plan-checks-spec.md` | None | uncovered, no finding (below) |

**Findings:** none. The help string is changed prose in a command's `Long` text, which is the same static-prose class as a stencil. I found no behaviour change that lacks a test row.

Tool calls (1):
1. Read `contracts/specs/specs_test.go` lines 14-68. My context held only the diff, not the test body, and I needed to confirm the non-empty-defaults loop.
