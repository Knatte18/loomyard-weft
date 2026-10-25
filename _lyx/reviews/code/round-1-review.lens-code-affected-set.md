```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens over the amendment (the merge of the parent's beam-window commit `afd030a47`).

**Per change, the set found and whether it is covered:**

| Change | Set found | Covered? |
|---|---|---|
| `runsStrandAddSpec(cmd, title)` new parameter | Only caller is `internal/orchcli/runs.go:66`, fed from the new field `c.runsWindowTitle`. That field is set in `internal/orchcli/cli.go:194` via `runsWindowTitleOf`. | Covered. The harness sets it (`start_test.go:165`), the title is asserted (`start_test.go:251`), and `runsWindowTitleOf` is table-tested (`start_test.go:360-367`). |
| `DobbyLines` gains a cells argument | Callers are `internal/beamcli/cli.go:110`, `internal/battencli/overviewdobby.go:80` and the test at `overviewdobby_test.go:93`. | Covered. All three pass cells. |
| `DobbyRunCells` / `NewDobbyRunCells` (moved) | Callers are `internal/beamcli/cli.go:91`, `overview.go:528` and the tests at `overview_test.go:351` and `:427`. The `beamcli` `doc.go` and the `battencli` `cli.go:24` doc comment name `DobbyRunCells`. | Covered. All callers and docs match. |
| `HubOrchs` / `PrimePaths` | Callers are `battencli/overview.go:357,365`, `tendercli/deps.go:392,398,414`, `cleanupcli/segments.go:64`, `battencli/beamland.go:183`. The `orchcli` `doc.go` describes both. | Covered. Nothing here is changed by the merge, and all callers pass resolved Locations. |
| `statusEntry` and `status` help text now say "a beam's window" | `beamcli/cli.go`, the `beamcli` `doc.go` and `docs/overview.md` (status and batten-overview paragraphs). | Covered. The three agree. The `orchengine` `doc.go:13` also says `beam: prime` / `beam: <slug>`. |
| `start.go` help text for the runs strand name | `orchengine/doc.go:13`, `docs/overview.md` orch row. | Covered. The title text is consistent. |
| `AnchorOverview` with a `Title` | `internal/loomcli/bootstrap.go:31` also uses `AnchorOverview` with no title. | Not a defect. It is a different strand, and the diff does not change it. |

**Findings:** none.

**Tool calls (2):**
1. `grep` for callers of the changed signatures (`runsStrandAddSpec`, `DobbyLines`, `overviewDobbyLines`, `newOverviewSource`, `NewDobbyRunCells`, `windowOrchPaths`). The manifest held only some of the call sites.
2. `grep` for `runsWindowTitle` / `AnchorOverview` / `beam: ` across the Go sources, and for the other docs that say "overview window". The manifest did not cover the tests or the other docs.
