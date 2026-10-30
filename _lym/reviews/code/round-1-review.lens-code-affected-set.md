```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines it touches, per change.

**Per-change list**

| Change | Set found | Covered |
|---|---|---|
| `[dobby notice]` line (stencil, `docs/overview.md`, orchcli doc, battenshed and dobbyengine docs, shuttleengine doc) | Orch stencil lines 43, 49, 51 and 90, `docs/overview.md` lines 454 and 711, `internal/orchcli/doc.go:78`, `internal/shuttleengine/doc.go`, the battenshed and dobbyengine `doc.go` files. A repo-wide grep for `batten notice` finds only the stencil and `battenshed/notice.go:24`. Stencil line 144 (Land or Worktree-Teardown) and the stencil's batten list name only notices batten keeps. | Covered. No moved notice is still named `[batten notice]` (D5). |
| Removed `InnerRunDeps` seams (`Activity`, `Sleeps`, `ReviveAPIError`, `NoticeQuiet`) and `AgentActivity` | `battencli/wire.go`, `shedrecipe/entries_batten.go`, `testkit/envkit/envkit.go` (`nilLegal` and `FullEnv`), `loomcli/wiring_seams_test.go` (`intentionallyNil`). A grep finds no remaining reference. | Covered. The `envkit` and `intentionallyNil` removals follow from the removed fields, so they are licensed. |
| Memory-record fields `APIRevive*` removed from `innerRunMemory` | `applyMemory` and `saveMemory` were updated. Old records carrying the keys still parse, because the records are tolerant. | Covered. |
| New marker `notice-episode` (`shedrun.NoticeEpisodeFile` and `NoticeEpisodeRel`) | `battenRunCommitPaths` and its test, `docs/overview.md:712`, and `cleanup.go:277`, which does `os.RemoveAll(shedrun.RunDir(...))`. | Covered. Cleanup removes the marker with the run directory (D3). |
| `notice_quiet_min` read by `runShedDuration` in `beamcli/dobbydeps.go`, validated but unused in `innerRunEntry` | `contracts/recipes/batten-recipe.yaml:78`, `entries_batten.go`, `entries_batten_test.go`. `dobbyengine` imports only `logger` and `pauserecord` beyond its existing set, both already imported in `state.go` and `loop.go`. | Covered, with F1 below. |
| `Deps.RunNotices`, `RunState.Notice` and the tick hook | `dobbyengine/doc.go`, `dobbydeps.go`, `battencli/dobbynotice.go`. | Covered. |
| `PATTERN.md` and `pattern/` | The diff does not touch them, and the directive's declared-changes list is empty. `PATTERN-dobby-starts-runs` and `PATTERN-agent-messaging` stay accurate. | Nothing to land. |
| Commit message | The claims match the diff. There is no `Pattern-Departure:` trailer, and none is needed. | Covered. |

**Findings**

- **F1 — LOW, consistency.** `internal/shedrecipe/entries_batten.go:122`, `internal/beamcli/dobbydeps.go:78-90`. The `innerRunEntry` doc says `notice_quiet_min` "validates ... non-negative". The `0 → 30` default is dropped and nothing now reads the key there, so an explicit `0` in a recipe reaches dobby as `Quiet = 0`. That silently disables the quiet notice, where before `0` meant the 30-minute default. The shipped recipe sets 30, so only a hand-edited zero changes behaviour.
  - Fix: either have `runShedDuration` treat a non-positive count as the fallback, or state in the `entries_batten.go` doc that zero disables the quiet notice.

No other member of the affected set is left wrong.

**Tool calls**
1. A grep for `batten notice` and for the removed seam names. My context lacked a repo-wide search for stale references.
2. `sed` of `batten-recipe.yaml` plus a grep for the notice terms and `PATTERN.md`. I needed the recipe comments and any other docs naming the notices.
3. A grep for the `RunDir`/`RemoveAll` cleanup path and the dobbyengine imports. I needed to confirm that cleanup removes the marker and that the new imports are already in use.
