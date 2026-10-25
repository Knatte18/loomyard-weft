```yaml
lens: code-pattern-fit
outcome: reported
```

Task: the code-pattern-fit lens over the Parent-Align merge `b244f3e14`, i.e. the changes brought in by `afd030a47`.

**Rule list**

- **`PATTERN-cwd-resolution`, declared text landed: held.**
  - `git diff afd030a47..HEAD -- PATTERN.md` shows the index entry carrying the declared text, including `(test)`.
  - `pattern/PATTERN-cwd-resolution.md` carries the matching bullets: own `.lyx-anchor`, `NamedWorktree`, only `fabricengine` writes or git-reads the marker, and the leaf import set.
  - The merge `b244f3e14` leaves `PATTERN.md` and `pattern/` untouched: a diff against its first parent shows neither file.
- **`PATTERN-cwd-resolution`, each worktree resolved through `lyxcwd` from its own marker: held.**
  - `orchcli.HubOrchs` (`internal/orchcli/live.go:19-50`) resolves each pair with `lyxcwd.ResolveWorktree`.
  - The overview command resolves the acting worktree with `lyxcwd.Resolve` (`internal/battencli/overview.go`, around line 500).
  - `DobbyRunCells.resolveChildPair` uses `lyxcwd.ResolveWorktree` (`internal/battencli/overview.go:226-237`).
  - `orchcli.PrimePaths` and `orchPaths` only read `location.AnchorPath()` of the `Location` they are handed (`internal/orchcli/paths.go:20-30`).
  - None of the merged code builds an anchor path by hand.
- **`PATTERN-cwd-resolution`, no production `Location` literal: held.**
  - A grep of non-test `.go` files for `lyxcwd.Location{` finds only `internal/testkit/locationkit/locationkit.go:12` and type-only uses (`[]*lyxcwd.Location{prime}`, `map[string]*lyxcwd.Location{}`). The scan's predicate treats those type uses as allowed, and `internal/testkit/` is skipped by the scan.
  - The only new literal from the merge is `&lyxcwd.Location{WorktreeName: name}` at `internal/battencli/overview_test.go:549`, in `TestOrchsAtOrBelow`.
  - Existing test files such as `arm_seed_test.go` already hold the same kind of literal and pass the tree scan, so test files are not scanned. The fixer report's claim holds.
  - The test literal sets only `WorktreeName`, which is all `orchsAtOrBelow` reads, so no path is derived from it.
- **`PATTERN-fabric-git`, `PATTERN-leaf-packages`, `PATTERN-lyx-temp-root`, `PATTERN-durable-vs-ephemeral-state`: held.**
  - The merged code runs no git; it only calls `fabricengine.List`, `ReadOrigin`, `WorktreePath` and `ConfigDirs`.
  - It imports nothing into `lyxcwd`.
  - It creates no temp files and names no `.lyx` or `_lyx` directory outside `lyxdirs`.
- **`PATTERN-cliwire-sole-wiring`, `PATTERN-cli-cobra`: held.**
  - The `beamcli` status verb gets its pause record from `hubgeom.PauseGeometry(prime)`.
  - No new standalone wiring was added, and the `internal/beamcli/cli.go` and `internal/orchcli` changes are within the existing cobra subtrees.
- **`PATTERN-spawn-observability`: held.** The merge starts no OS process.
- **`PATTERN-daemon-wakeups`, `PATTERN-pane-text-injection`, `PATTERN-agent-name`: no applicable change.** The reed strand window title is a display string only.
- **Text-only overlaps (`docs/overview.md`, `internal/beamcli/cli.go`, `internal/beamcli/doc.go`): held.**
  - `git diff 4e7f23e48 b244f3e14` shows exactly the parent's beam-window sentences: "a beam's window", `DobbyRunCells`, "time in stage".
  - No sentence of the task's anchor text was dropped, and none contradicts the parent's.
  - The merge changed the module doc and the module table together with the code, as CLAUDE.md "Docs land in the same commit" asks.
- **Commit messages and `Pattern-Departure:` trailers: held.**
  - No commit in `afd030a47..HEAD` carries a `Pattern-Departure:` trailer.
  - Every commit ends with the required `Co-Authored-By` line.
  - The `b244f3e14` merge message has no body.
- **Fixer report claims: held.**
  - The `## Fixes` section is empty.
  - The merge's diff against `4e7f23e48` is the parent commit's 21 files only (`afd030a47~1..afd030a47` shows the same 21), so nothing outside the amendment is touched.

**Findings:** none.

**Tool calls**
1. `git diff afd030a47..HEAD -- PATTERN.md pattern/PATTERN-cwd-resolution.md`, plus `sed` of `internal/lyxcwd/enforcement_test.go` lines 925-950 and 200-215. I needed the landed PATTERN text and the scan's skip rules, which the manifest lacked.
