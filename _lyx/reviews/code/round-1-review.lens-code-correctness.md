```yaml
lens: code-correctness
outcome: reported
```

**Task:** check the correctness of the code the Parent-Align merge `b244f3e14` brought in, with one concrete case per branch.

**Result:** no correctness findings.

| Function or branch | Case run | Verdict |
|---|---|---|
| `orchcli.HubOrchs` (`live.go:19`) | The hub lists no pairs, so only the prime is returned. | Correct. |
| `HubOrchs`, pair loop | `ResolveWorktree` fails (the pair's own marker is unreadable), so the pair is skipped with a warning. | Correct. The doc comment states this, and the reader has to keep working under a pause. |
| `HubOrchs`, `ReadOrigin` | It fails, or the origin has no beam mark, so the pair is skipped. | Correct. |
| `HubOrchs`, main and prunable entries | They are skipped, and the prime is added once through `resolvePrime`. | Correct. |
| `runsWindowTitleOf` (`runs.go`) | `beam=false` gives `beam: prime`, and `beam=true` with `x` gives `beam: x`. | Correct. `cli.go:155-161,194` derives `beam` from the origin record, and a failed origin read already returns an error. |
| `runsStrandAddSpec(cmd, title)` | The title goes into `Display.Title`. | Correct. |
| `DobbyRunCells.resolveChildPair` | The first call resolves with `ResolveWorktree` and caches the pair. A failure is not cached and is retried on the next poll, logged at debug. | Correct. |
| `poll`: pair-cache eviction | A removed worktree directory evicts its cached pair before `childStatus` runs, so a recreated pair resolves again against its own marker. | Correct. |
| `poll`: cells for a run that is not in Run-Shed | The child status and recipe are not read, and the cells are built from the prime's batten records. | Correct. |
| `Of`: pause record | A read error gives an empty record and the run is not paused. A covering entry sets `PausedAt`. | Correct. |
| `windowOrchPaths` | The prime's location gets an empty label, and a beam gets its slug as its label. `PrimePaths` takes each resolved location's own anchor. | Correct. |
| `orchsAtOrBelow` and `atOrBelow` | The prime's window returns all locations. A cycle (`loop-x` ↔ `loop-y`) returns an error, so that location is left out. A `parentOf` error leaves it out. A chain ending at the prime gives false. | Correct. |
| `newOverviewSource` | The prime's label is `""`. `PrimePaths(worktree)` uses the acting worktree's own resolved location. | Correct. |
| Overview command wiring | The cwd goes through `lyxcwd.Resolve` for the acting worktree and `ResolveBattenPrime` for the prime. A `PauseGeometry` failure ends the command with an error envelope. A single `cells` value is used by one frame loop only, and the type is documented as not safe for concurrent use. | Correct. |
| `beamcli.dobbyStatus` | One `DobbyRunCells` serves every worktree's `view.Runs`. `PauseGeometry` runs once, and its failure aborts with a wrapped error. The cells map is keyed by slug, and batten records sit under the prime, so slugs do not collide across dobbies. | Correct. |

**Findings:** none.

**Tool calls (2):**
1. Read `internal/battencli/overview.go:372-420` and the `beam` origin in `orchcli/cli.go`. My context held only the head of `orchsAtOrBelow` and had no `atOrBelow` or `beam` derivation.
