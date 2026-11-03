# Card 4 — status-strand-second-overview

**Create:**
- `internal/loomcli#statusStrandOrder`

**Edit:**
- `internal/loomcli#statusStrandAddSpec` -> `func statusStrandAddSpec(cmd, recipe string) reedengine.AddSpec`
- `internal/loomcli#loomCLI.ensureStatusStrand`
- `internal/loomcli/doc.go`
- `docs/overview.md`
- `internal/loomcli#TestStatusStrandAddSpec`
- `internal/loomcli#TestSmokeStatusStrandAcrossDriverSeeds`

**Uses:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#windowID`
- `internal/loomcli#strandPane`
- `internal/loomcli#statusStrandPane`
- `internal/loomcli#seedLLMDriver`
- `internal/loomcli#tmuxBinaryPath`
- `internal/loomcli#statusSidecarPath`
- `internal/orchcli#RunsStrandOrder`

**Intent:**
D3 and D5: the loom status strand always takes the overview anchor, one place after the runs strand, named for its recipe.

- `statusStrandOrder` is the status strand's overview order key, derived from `orchcli.RunsStrandOrder`, so the checklist window sits right after the beam overview.
- `statusStrandAddSpec` takes the run's recipe name in place of the session's strands and always returns `render.Display{Anchor: render.AnchorOverview, Order: statusStrandOrder, Title: recipe}`; the runs-strand branch with its own window and `KeepViewer` goes, so it no longer reads the session's strands. Its doc comment says so.
  The title comes from the run's own seed, never a branch on a recipe name (`PATTERN-no-hard-coded-choice`).
- `ensureStatusStrand` passes `c.recipe`, which `armAt`'s `resolveRecipe` records before `start`'s body runs and which equals the seed's recipe, since a start with no seed writes it with the same `resolveSeedRecipe` answer in `seedAndCommitBootstrap` before step 4.
  Its keep, replace or add choice is unchanged, so a session from before this change heals at the pawl run's next start: a status strand in an own window with no sidecar of this build is replaced into the overview window, and a missing one is added.
- `doc.go` (line 62's paragraph): the status strand runs `lym loom checklist` in an overview window named for the run's recipe, placed right after the runs strand's overview window in a beam's session; the "or, in a beam's session where the beam's `runs` strand holds that window, in a window of its own that never takes the viewer" clause goes.
- `docs/overview.md`: the loom entry's `start` sentence and the glossary's "the bootstrap" entry each drop "or in a window of its own that never takes the viewer when a beam's `runs` strand holds the overview" and say the status strand takes an overview window named for the run's recipe, right after the runs strand's in a beam's session.

Tests:
- `TestStatusStrandAddSpec` (tier0) drops its strands field and gains a recipe field: rows `PawlRecipeTitlesTheWindowPawl` (`pawl`) and `TapestryRecipeTitlesTheWindowTapestry` (`tapestry`) each want `Display{Anchor: AnchorOverview, Order: statusStrandOrder, Title: <recipe>}`, with the name, command and `IfAbsent` assertions kept; each row also asserts `got.Display.Order > orchcli.RunsStrandOrder`, so a status key not after the runs strand's fails at tier0.
- `TestSmokeStatusStrandAcrossDriverSeeds` (tier2): its two `statusStrandAddSpec("sleep 3600", nil)` calls pass `shedrun.RecipeTapestry`.
  The subtest `start beside a runs strand adds the status strand in its own window` becomes `start beside a runs strand puts an own-window status strand into the overview window after it`: after reed up it adds the runs strand with `Display{Anchor: AnchorOverview, Order: orchcli.RunsStrandOrder, Title: "beam: smoke"}` and a status strand under `statusStrandDisplayName` with `Display{Anchor: AnchorOwnWindow, KeepViewer: true}`, the placement an older lym gave it, and writes the status sidecar at `statusSidecarPath` with `statuspane.WriteSidecar`, naming that strand's guid and another build (`statuspane.BuildIdentity{Path: "/older/lym", Size: 9, ModTime: 9}`), the D5 case of a sidecar naming another build; after `lym loom start --no-attach` it wants one status strand, a guid other than the pre-added one, its window distinct from the runs window, carrying `@lym_overview` 1, named `tapestry` (the recipe `seedLLMDriver` seeds), and its `#{window_index}` one above the runs window's, each read with `display-message -p` on `tmuxBinaryPath` as `windowID` does.
  The file's header comment says that start puts the status strand in the overview window after the runs strand.

**ImpactSummary:** loomcli's status strand only: `statusStrandAddSpec`'s callers are `ensureStatusStrand` and the two tests listed; `lym loom step` adds no strand and is unaffected; a task run's checklist window is now named for its recipe, `tapestry` or `darn`, as the record's auto-mode assumption states.

**Commit:** 4: loomcli: put the status strand in an overview window after the runs strand, named for its recipe
