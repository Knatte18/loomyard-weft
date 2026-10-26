```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set — what the diff's changes force outside the lines they touch.

**List (per change, set found, covered or not)**

- `paneProcessAge` (linux) now reads the tick rate via `unix.Auxv` and `clockTicksFromAuxv`.
  - Set found: the sole caller, `lock.go:78`, which assigns `paneProcessAge` to `paneAge`.
  - The windows stub (`proctree_windows.go:56`) keeps its signature.
  - `processAge(uptime, stat, ticksPerSecond)` is unchanged and keeps its existing `TestProcessAge`.
  - Covered.
- cgo removed from `internal/reedengine`.
  - Set found: a grep of Go, md, yaml and sh files for sysconf, `_SC_CLK_TCK` and cgo near reedengine or proctree finds no stale mention beyond the new comments.
  - `internal/reedengine/doc.go:309` mentions only `paneProcessAge`, not the cgo mechanism.
  - Covered.
- New `golang.org/x/sys/unix` import in reedengine.
  - Set found: `go.mod:15` already lists `golang.org/x/sys v0.45.0` without an indirect marker, so `go.mod` and `go.sum` need no change.
  - `PATTERN-leaf-packages` does not bind reedengine, per the decision record.
  - Covered.
- New no-cgo rule, `cmd/lyx/nocgo_test.go`.
  - Set found: `pattern/PATTERN-gate-slot.md` (new bullet beside `-trimpath`, overview line unchanged) and `internal/gateslot/doc.go` (two new lines).
  - `CLAUDE.md`'s "Build: cgo" section still says lyx needs cgo for the tree-sitter grammars.
  - That stays true, since the guard scans the module's own files only.
  - Covered.
- New `//lyx:guard` test file.
  - Set found: `PATTERN-test-speed.md:19` and `:77-82` require one `tierN` tag, with `//lyx:guard` directly above the `func Test…`.
  - The diff has `//go:build tier0` and the guard directly above `TestNoCgo_NoFileOfTheModuleImportsC`.
  - Covered.

No findings.

**Tool calls**
1. One grep over `.go`, `.md`, `.yaml` and `.sh` files for sysconf, `_SC_CLK_TCK`, cgo and reedengine/proctree mentions, plus `x/sys` in `go.mod`. My context lacked any repo-wide search for stale references or the dependency's direct/indirect status.
