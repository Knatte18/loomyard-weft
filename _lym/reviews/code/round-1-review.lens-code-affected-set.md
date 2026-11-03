```yaml
lens: code-affected-set
outcome: reported
```

Task restated: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set I found:**

1. **`render.Display.Order` (new field, `order,omitempty`).**
   - Readers and writers: `overviewWindowKeys`, `refuseWindowMove`, `runsStrandAddSpec` and `statusStrandAddSpec`. All are covered.
   - Tests: `types_test.go` covers the decode of a missing key as zero and the round trip. Covered.
   - Persistence: it goes through reed.json under the existing tolerant-record decode. I found no other reader or writer that needs the key. Covered.

2. **`overviewWindowInsertArgs` signature.**
   - The only production caller is `openOwnWindowLocked` (`spawn.go:283`). It is updated.
   - The tests use the new signature (`windoworder_test.go`). Covered.

3. **`agentWindowPanes` return type (`overview` is now a set).**
   - The only caller is `relevantWindow` (`ownwindow.go:457`). It is updated.
   - `relevantWindow` has three callers (`ownwindow.go:542`, `:564`, and via `selectRelevantWindowLocked`). Their signatures are unchanged. Covered.

4. **`followTargetWindow`.**
   - Its one caller is `follow.go:108`.
   - The `lym reed follow` help text, the `doc.go` FollowSession paragraph and the `reedengine/doc.go` focus-rule paragraph are updated. Covered.

5. **Removal of the add-time refusal of a second overview strand.**
   - The `addStrandLocked` loop, the `ReplaceStrand` doc sentence and `TestAddStrandLocked`'s `SecondOverviewUnderAnotherNameRefused` subtest are removed.
   - The `doc.go` "Overview anchor" and focus paragraphs are rewritten.
   - A search for "one overview", "one-per-session", "the session's one" and "already holds the overview" in production code, docs and stencils found no survivor.
   - `reedcli/add.go` still refuses the overview anchor as engine-only, which D1 says to keep. Covered.

6. **`refuseWindowMove` order refusal (new message).**
   - The refusal reaches only `UpdateStrand`.
   - The way forward it names is "have its spawner add it again", not `lym reed add`. This is consistent with `reedcli/add.go` refusing the overview anchor as engine-only.
   - It is covered by `TestUpdateStrandLocked`.

7. **`ensureRunsStrand` stops looking for a loom status strand.**
   - The `loomengine` import and the `LoomStatusStrandName` comment are updated.
   - `orchcli/doc.go`, `orchengine/doc.go` and the `docs/overview.md` batten and orch entries are updated.
   - The one caller is `beamUp`. Covered.
   - Out-of-range note: `LoomStatusStrandName` is also used by `loomcli`. That is not stale.

8. **`statusStrandAddSpec(cmd, recipe)`.**
   - Its callers are `ensureStatusStrand` (`sharedbootstrap.go:300`) and the two smoke-test call sites. All are updated.
   - `loomcli/doc.go` and the two `docs/overview.md` spots (loom `start` and the glossary "bootstrap") are updated.
   - The smoke-test header is updated.
   - A search for "KeepViewer", "never takes the viewer" and "window of its own that never" found no stale statement about the status strand. `render/types.go:56` and `shuttlecli/run.go:139` concern agent strands only.
   - The `loomcli/driverspec.go:45` comment, "an agent window of its own, after the overview", is still true. Covered.

9. **PATTERN, commit messages and stencils.**
   - The diff touches no `PATTERN.md` or `pattern/` file. The directive declares none, so there is nothing to land.
   - Commit messages carry no `Pattern-Departure:` trailer, and I found no difference from the answer key that would need one.
   - `PATTERN-named-constants`: `RunsStrandOrder` is declared once in `orchcli/runs.go` and `statusStrandOrder` is derived from it once. Compliant.
   - `PATTERN-no-hard-coded-choice` and `PATTERN-loom-contract`: the title comes from `c.recipe`, with no branch on a recipe name. Compliant.
   - No stencil is touched, and I found no stencil that mentions the overview window.

10. **Doc statements of one overview strand per session.**
    - I found no survivor in reed, orchcli, orchengine, loomcli, loomengine or `docs/overview.md`.

**Findings:** none. Every member of the affected set I found is updated, and no stale statement of the old behaviour survives.

**Tool calls (4 of 6):**
1. A repo-wide grep for overview and loom-status wording. It returned mostly unrelated plan-overview hits, which showed the first search was too broad.
2. A grep scoped to the reed, orch, loom and batten packages and the docs. My context lacked the surviving-wording check.
3. A grep for `KeepViewer` and "never takes the viewer" wording. My context lacked this check for stale comments about the status strand.
4. A grep for index-0 and "first window" assumptions in reed. My context lacked this check for code that still assumes a single overview window.
