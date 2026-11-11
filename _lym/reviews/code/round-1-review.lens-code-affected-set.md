```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change the diff makes, I looked for what reads, calls or asserts it outside the lines it touches. The change-by-change list comes first, then the findings.

**Change by change**
- **claude-file now renders `CLAUDE.local.md` alone:**
  - Set found: `target.go` table and `ReRender`, `fileText`, the `seam.go` `TookIn` path matching, the `shipped.json` entries, tests that name claude-file, and the fabric exclude template.
  - `TookIn` selects targets by `ReRender` path and `Render` skips shared targets, so agents-file needs no re-render paths of its own.
  - The two shipped entries keep reaching Claude only, as before, and the exclude list already carries both rendered files.
  - Covered.
- **The new shared `agents-file` target:**
  - Set found: `LookupTarget` and `TargetFiles`, `ValidateEntry` through `TargetNames`, `reachesFileTarget`, `Show`, the overview `targetReach`, `lym heddle list` and `set --help`.
  - All of them know the shared target. `Show` refuses it with a way forward, and no consumer lists targets elsewhere in the repo.
  - Covered.
- **The unversioned-store migration:**
  - Set found: `Load`, `Decode` and its three replayer callers in `replayer.go`, `ReadLayers`, and the override file through `LoadOverrides` and `SaveOverrides`.
  - The override layer goes through `loadFile` without `upgradeStore`, so its targets stay unmigrated, but only entry text is read from it by slug, so that is harmless.
  - `Save` writes the version through `MarshalJSON`.
  - The fabric replay test encodes its heddle file through `Entry` and `Store` marshalling, so it is versioned, and the replayer decodes all three sides through `Decode`.
  - Covered.
- **Documentation surfaces:**
  - Set found: `set --help`, `lym heddle list` (`file_targets`), `HEDDLE.md` via `overview.go`, `heddleengine/doc.go` and `heddlecli/doc.go`.
  - Grep for the old "renders into both" wording found no stale text outside the diff.
  - Covered.
- **PATTERN rules:**
  - `PATTERN-config-layers` is not engaged, because `heddle.json` is not a `configengine` layer. `heddlecli` has no pattern entry that applies.
  - `PATTERN.md` shows no declared change, and no commit carries a `Pattern-Departure:` trailer.
  - Covered.

**Findings**
- **F-A, LOW, consistency:** `pattern/PATTERN-lym-files.md:33` says "The rendered files are `CLAUDE.local.md` and `AGENTS.override.md`." That stays true, but it does not mention that the targets are now per-family.
  - I read this one line without its surrounding paragraph.
  - It is only a candidate; I would not require a fix unless the surrounding paragraph says claude-file renders both.
- No other member of the affected set is left wrong.

**Tool calls (3)**
1. Read `seam.go` lines 30–90 and `shipped.go` lines 60–160, because the manifest lacked `TookIn` target selection and the `ReadLayers` and override paths.
2. Read `PATTERN-config-layers.md` and `PATTERN-heddle-instructions.md`, and grepped for `loadFile` and `saveFile` callers, to check PATTERN fit and whether the override file bypasses the migration.
3. Grepped the repo for "AGENTS.override" and the old "both" wording, and for "targets" in `verbs.go`, to find stale prose and other consumers.
