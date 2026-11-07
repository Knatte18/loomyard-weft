```yaml
lens: code-tests
outcome: reported
```

**Per behaviour, the test that pins it:**

- **Every filed task is draft, in every shape:** pinned by `TestFileRaddle` (`internal/boardengine/raddlefile_test.go:171`). The shared loop checks that each `KindTask` entry has status `draft`, across the tapestry, tasks, project and grouping-project rows. Releasing a part would fail it.
- **No `Release` flag or release step in the filing:** the compiler enforces the flag's absence. `TestRaddleFile` (`internal/loomshed/raddlefile_test.go:242`) pins the beam step order `commit, file, create_beam, notify, end_entry`, and `inOrder` pins the other shapes.
- **Filed notice says the tasks wait as drafts:** pinned by the `strings.Contains` assertion in `TestRaddleFile`, which checks `every filed task is draft until the operator releases it`. It runs for every shape.
- **Notify and end-entry halt rows after the `release` row's removal:** pinned by the remaining halt table (`create_beam`, `notify`, `end_entry`).
- **`Board.ReleaseTasks` and the `raddleFiled` result path removed:** the compiler enforces this. A repo-wide grep (it ran before the fork) finds no leftover reference. The "already filed" early return in `FileRaddle` (`return nil, nil`) is pinned by the existing resume subtests of `TestFileRaddle`, which are unchanged apart from dropping `Release`.
- **`Release` seam removed from `RaddleFileDeps`:** `internal/battencli/wire_seams_test.go`, `internal/shedrecipe/entries_raddle.go` and the two fixtures are consistent. A stale `intentionallyNil` entry would fail that table, and the diff removes it.
- **Optional `order_departures` key parses, with `parts`, `record_order` and `cost`:** pinned by the `OrderDepartures` row of `TestParse_OptionalKeys` (`internal/raddleparser/proposal_test.go:77`). It is exact, via `reflect.DeepEqual`.
- **`order_departures` is optional, and its omission changes no hash:** the `PatternClusters` row of the same test parses with the key absent. Because `omitempty` is set, an absent key marshals identically.
- **An unknown key inside an `order_departures` entry is refused:** no row covers it. `rejectUnknownKeys` is generic (reflection over slices and structs, `proposal.go:176-200`), and the existing rows at `proposal_test.go:29-31` already pin that mechanism at three depths. A per-key row would only repeat that coverage, so I raise no finding.
- **The `packages-shared` check still refuses two unordered parts on one package:** `check.go` is untouched and its existing check tests are unchanged.
- **Approval table with a dependencies column, order departures named, tasks land as drafts:** this is Go-built prose in `raddleModeRules` (`internal/loomengine/raddletable.go:36-41`). `internal/loomengine/raddletable_test.go:84,96` only compares the prompt against `raddleModeRules(...)` itself, so it pins nothing about the wording. That text is static prose and exempt, and I raise no finding.
- **Stencil and spec wording:** static prose, exempt.

**Findings:** none. No behaviour the diff adds or changes lacks a test row.

**Test-economy observation, not a finding:** `TestParse_OptionalKeys` folds `TestParse_PatternClusters` into a table, so the old test's coverage is kept as a row. The removed `TestReleaseTasks` and the removed beam row of `TestFileRaddle` covered behaviour that no longer exists. No redundant test was added.

**Tool calls:**
1. `sed` of `internal/raddleparser/proposal.go:150-260`, plus a grep of `proposal_test.go`. My context lacked `Parse`, `rejectUnknownKeys` and the existing unknown-key test rows, which show that a new nested struct is covered generically.
