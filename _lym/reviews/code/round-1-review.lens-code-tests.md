```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour, the pinning test**

| Behaviour changed | Pinning test |
|---|---|
| `PATTERN.md` shape: entry names unique, `PATTERN-[a-z0-9-]+`, a background link resolving to a file | The real-file check in `internal/pattern`, run under `go test`. It is the existing mechanical gate, and the diff leaves it unchanged. |
| Overview at or below `MaxOverviewBytes` | `Check`'s cap check. Same existing coverage. |
| Headroom line | `internal/pattern/headroom_test.go`. The diff does not touch it. |
| ID rewrites in Go comments, doc comments and `t.Errorf` strings (about 40 files) | None needed. They change only text and no logic. No test asserts the old or the new strings, since the grep for old IDs found none and no test cites the new IDs. |
| `planparser` finding `Detail` now says `PATTERN-fabric-boundary` (`internal/planparser/exportname.go`, `vocabname.go`) | None. My grep of `internal/planparser/*_test.go` for `breaks PATTERN` or `PATTERN-fabric` found nothing. The text is a message string, and the rubric exempts prose. The checks themselves (`CheckExportName`, `CheckFabricVocabularyName`) are unchanged and keep their existing tests. |
| `testtiming:keep` comments in `contracts/stencils` tests | None needed. The comment text changed and the tests did not. |
| Cluster background files (`pattern/*.md`) and the `docs/overview.md` and `contracts/specs/loom-plan-checks-spec.md` prose | Exempt as static prose. |

No finding. The diff adds no new test and no behaviour a test could pin. It adds no test-economy violation either: it adds no test, no per-helper test and no assertion on prose.

**Tool calls**
1. `grep` in `internal/planparser/*_test.go` for `breaks PATTERN|PATTERN-fabric`. I needed it because my context held no planparser test content, and it settled whether any test pins the rewritten `Detail` text.
