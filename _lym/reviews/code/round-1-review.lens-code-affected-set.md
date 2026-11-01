```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set — what the diff's changes force outside the lines they touch.

**Per change: set found, covered or not**

1. **`ensureParentDobby` / `battenCLI.dobbySpawn` → `pairStart`**
   - Set: `internal/battencli/cli.go` and `cli_test.go`. The grep shows no other reader of `dobbySpawn`.
   - Covered.
   - `anchor.go:530` says `dobby_start_error` carries the way forward "lym beam start". It stays correct.

2. **`startBeamDobbyAlone` removal, now `orchcli.BeamUpWorktree` in pawl `StartDobby`**
   - Set: `loomcli/pawlwiring.go`, `loomcli/doc.go` and `pawlengine/{deps.go,doc.go}`. All were updated.
   - The grep finds no remaining reference to `startBeamDobbyAlone`. Covered.

3. **Anchor move `EnsureDobbies` (`dobbyengine.EnsureLive` → `beamStart` child)**
   - Set: `anchor.go` doc, `fabriccli/doc.go`, `anchor_test.go` and `anchor_integration_test.go:197`.
   - The doc comments and the call-order row were updated. Covered.
   - The `anchor_integration_test.go:197` assertion on `dobby_start_error` is unchanged. In a test binary `beamStart` returns an error and the assertion still passes.

4. **New `PATTERN-dobby-orch-pair` entry**
   - The entry is in `PATTERN.md:40`. The grep finds no `pattern/PATTERN-dobby-orch-pair.md`, and the entry has no `(test)` marker.
   - See F1.

5. **`orchcli/doc.go:11`**
   - Not covered. See F2.

6. **`beamcli/doc.go` (lines 107 and 141)**
   - It describes `SpawnDetached` as the dobby launcher. That is still true inside `EnsureLive`, so no change is needed.

**Findings**

- **F1 — LOW, consistency.** `PATTERN.md:40` is a new entry with no `(test)` marker and no `pattern/PATTERN-dobby-orch-pair.md` background file. Every other entry about a verb-level rule either carries one or is a short prose-only rule; the sibling `PATTERN-dobby-starts-runs` (line 39) also has neither, so this is a nit-level gap rather than a rule breach. The entry names a rule that is checkable by grep (`EnsureLive` or `SpawnDetached` callers outside `orchcli`), but no test pins it. Evidence: `PATTERN.md:39-40`, and the `ls pattern | grep -i dobby` output, which is empty.

- **F2 — LOW, consistency.** `internal/orchcli/doc.go:11` still says "`BeamUp` starts this worktree's dobby alone through `dobbyengine.EnsureLive`". That is accurate for `BeamUp`'s internal step, and the new `PATTERN-dobby-orch-pair` entry now forbids "alone". Reword it to say the dobby start is the first step of the pair, so the doc does not contradict the entry. Evidence: `orchcli/doc.go:11`, with the new line 13 beside it and `PATTERN.md:40`.

- **F3 — MEDIUM, scope.** `PATTERN.md:40` says "through `orchcli.BeamUp`, or `orchcli.BeamUpWorktree` or a `lym beam start` child". The `hubgeom/dobbygeom.go:115` `Spawn: dobbyengine.SpawnDetached` wiring is a lone-spawn route that the rule's own "never `dobbyengine.SpawnDetached` alone" wording would catch. The entry has no exemption for `hubgeom`'s seam wiring or for `orchcli`'s own `ensureDobbies` (`start.go:338`). Add the carve-out, "bar the `hubgeom.DobbyStart` seam and the `BeamUp` step", so the rule is not violated by its own callers. Evidence: `hubgeom/dobbygeom.go:115`, `orchcli/start.go:338`, `PATTERN.md:40`.

No stencil files reference the changed behavior, and I searched `.md` files with the same grep. The commit messages carry no `Pattern-Departure:` trailer, and none is needed.

**Tool calls**
1. A grep over `.md`, `.go`, `.yaml` and `.tmpl` for `EnsureLive`, `startBeamDobbyAlone`, `dobbySpawn`, `EnsureDobbies` and `dobby_start_error`, plus a listing of `pattern/` for dobby files and a grep of `PATTERN-owner-machine.md`. My context lacked the full reference set for these symbols and the `pattern/` directory listing.
