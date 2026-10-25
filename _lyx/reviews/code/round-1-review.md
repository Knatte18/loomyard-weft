---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/battencli/overview.go:226
    summary: DobbyRunCells caches a resolved task worktree Location for as long as its directory exists, so a later anchor move of that pair leaves the cached AnchorRel stale and the child status and seed are read at the old anchor
    fix: In internal/battencli/overview.go, make `poll` drop the cached pair whenever the run is not in Run-Shed, by calling `delete(r.pairs, slug)` in the branch where `run.Batten.Status.CurrentProducer` is not `battenrecipe.NameRunShed`, so the next Run-Shed read resolves the pair again through `lyxcwd.ResolveWorktree` and reads its current marker. Update the `pairs` field comment to say the entry lives while the run stays in Run-Shed. Add one row to the `DobbyRunCells` test in overview_test.go that changes the pair's marker between two Run-Shed spells and asserts the second read uses the new anchor.
    origin: handler
  - id: F2
    severity: LOW
    class: design
    location: internal/orchcli/cli.go:194
    summary: The wiring `c.runsWindowTitle = runsWindowTitleOf(location.WorktreeName, beam)` has no test, since the strand-spec test sets the field by hand and the title test calls only the pure helper
    fix: Add one row to the existing `orchCLI` start test in internal/orchcli/start_test.go (or its wiring test) that builds the CLI through its command setup for a prime and for a beam and asserts the `runs` strand is added with `Display.Title` "beam" then a space-separated prime label or the beam's slug, as `runsWindowTitleOf` returns them; do not add a second test of `runsWindowTitleOf`.
    origin: lens:code-tests
---

The amendment is the Parent-Align merge `b244f3e14` of the parent commit `afd030a47`.
The merge has no edit of its own: its diff against the first parent is the parent commit's 21 files, and the fixer report's empty `## Fixes` section is borne out.

### [LOW] Cached task worktree Location outlives an anchor move

**Location:** internal/battencli/overview.go:226
**Issue:** `DobbyRunCells.resolveChildPair` resolves a task worktree once and keeps it until the directory disappears (`poll` deletes the entry only when `os.Stat` of the worktree fails).
The anchor change adds `lyx fabric anchor --set`, which moves a pair's anchor with its directory in place, and D7 permits it once the pair has no live run, strand or held driver lock.
A long-running overview of the prime that saw a run of the pair in Run-Shed, then saw the run end, then saw the pair moved and a new run start in it, reads the child's status file and seed at the old anchor and shows the child as not found.
The decision record's D2 and D3 ask that a worktree's Location come from its own marker; a cache keyed on directory existence does not follow a marker change.
The cache is the parent's own, but the anchor change is what makes the cached value stale.

### [LOW] Window-title wiring is untested

**Location:** internal/orchcli/cli.go:194
**Issue:** `runsWindowTitleOf` is pinned by its own table and the strand spec by a test that injects the title, so a wrong worktree name or `beam` flag passed at `cli.go:194` fails no test.
This is behaviour the merge adds with no coverage, which the rubric's test economy lists as a finding.

## Rejected

- Code-tests NIT, `HubOrchs` skip-and-warn paths untested: `HubOrchs` is not in the merge's diff, so it belongs to the settled side.
- All checks of the focus directive hold, so they raise nothing: each merged reader resolves its worktree through `lyxcwd.ResolveWorktree` or `lyxcwd.Resolve` and takes `AnchorPath()` from that Location (`internal/orchcli/live.go`, `internal/battencli/overview.go`, `internal/orchcli/paths.go`); the only new `lyxcwd.Location{` literal is in `internal/battencli/overview_test.go`, which the D3 scan does not cover; the text overlaps in `docs/overview.md`, `internal/beamcli/cli.go` and `internal/beamcli/doc.go` keep the parent's beam-window text with no dropped or contradictory sentence; and the declared `PATTERN-cwd-resolution` change is in the branch's `PATTERN.md` and `pattern/` diff.
