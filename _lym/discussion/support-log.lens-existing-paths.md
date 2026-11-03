```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: trace the code paths that already write or read the state D1 to D5 change, and say whether each decision survives them.

**Paths traced, per decision**

- **D1, relaunch through `openOwnWindowLocked`** (Resume replay, reconcile relaunch, ReplaceStrand's add): survives. Overview placement ignores `previousWindow` and always goes through `overviewWindowInsertArgs`, so D1's new rule covers relaunch once that function changes. The `sessionWindow` listing has no overview flag and no order key, and the record's plan note names this.
- **D1, agent window placement through `ownWindowInsertArgs`** (previous window, else before shell, else after the highest visible window): survives. A relaunched agent window goes after its old window, which is already behind the overview windows.
- **D1, shell window repair through `ensureShellWindowLocked`**: survives. It only moves the shell window behind the highest visible window.
- **D1, `AddStrand` refusal (strand.go:404) and ReplaceStrand's third-strand rule**: removed per D1. Survives.
- **D1, `UpdateStrand(guid, display)` (strand.go:700) and `refuseWindowMove`**: breaks, silently. An update can change the new order key without moving the window.
- **D1, `lym reed add --anchor overview` and shuttleengine's `AddStrand` callers** (open.go:85, run.go:633), which pass a spec's display through: breaks D1's bound.
- **D2, `agentWindowPanes`, `relevantWindow`, `followTargetWindow`**: each keeps one `overview` variable that is overwritten as it loops, so the pick follows iteration order. Covered by D2, survives once changed.
- **D2, loom start's follow re-arm after `ensureStatusStrand`** (start.go, "selects the run's relevant window"): survives. In a beam session the viewer now lands on the beam overview, not the checklist, which matches D2's rationale.
- **D3, `resolveRecipe`**: survives. It runs in the pre-run before seeding, and falls back to tapestry when there is no seed yet. A pawl run is seeded by `lym beam create`'s batten pass before its loom start, so the title reads the seed's recipe.
- **D3/D5, `resolveStatusStrandAction`**: survives. It keeps only a live strand whose sidecar names the current build, so a deploy of this change replaces a legacy own-window status strand. Until then that legacy strand counts as an agent window in `relevantWindow` and follow, within D5's stated bound.
- **D4, `ensureRunsStrand`'s keep and replace of the runs strand through ReplaceStrand**: survives. The replacement window reopens with key 0, before the checklist.
- **D5, an older-build lym process still alive on reed state** (a dobby or a pane's pinned binary doing reed ops): partly unverified.

**Findings**

- MEDIUM, D1 bound: "every overview spawner adds under explicit role" is false for existing entry points. `lym reed add --anchor overview` with no explicit name gets the default role `strand`, and a held role is numbered `-N`, not refused (reed doc.go, naming paragraph). Shuttle's `AddStrand` callers pass the spec's display through too. With the cap gone, these paths add any number of overview windows. The bound should say so: either no cap from those paths, or keep a refusal there.
- MEDIUM, D1: `UpdateStrand` takes a whole `render.Display`, and `refuseWindowMove` checks only the anchor and group. An update that changes the order key is accepted and the window stays where it is, so the window order no longer matches the keys until the next relaunch. The record says nothing about updates. It should either refuse an order-key change on update, as it refuses an anchor move, or re-place the window.
- LOW, D1/D5: an older lym binary still running reed ops refuses a second overview strand and places any overview window it relaunches at the lowest index. It may also drop the unknown order key when it rewrites reed.json; I did not check whether reed state keeps unknown keys under `PATTERN-tolerant-records`. D5's bound covers older records but not older live processes.
- LOW, D2: `agentWindowPanes` returns one overview id, taken from whichever bound pane comes last. D2 should name it beside `relevantWindow`, because that is where the single-overview assumption actually lives.

**Tool calls**

- grep for the overview anchor and marker readers across internal and cmd: my context held only the reed doc grep, not every reader outside it.
- Read of spawn.go 250–300, ownwindow.go 424–470 and follow.go 58–86: I needed the placement and fallback bodies, and my context held only their comments.
- grep of loomcli/start.go plus arm.go 120–145: to check whether the recipe is resolved before the status strand is ensured, which my context did not show.
- grep of beamcli/create.go, `UpdateStrand` callers and `AddStrand`/`ReplaceStrand` callers: to find other writers of overview displays, which my context lacked.
