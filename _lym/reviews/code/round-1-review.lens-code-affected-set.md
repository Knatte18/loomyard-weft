```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change, the set it forces outside the touched lines, covered or not.

**Per-change list**

1. **`arras` recipe name (`shedrun.RecipeArras`, `RecipeNames`)**
   - Covered: `shedcli` arming table, `loomrecipe` (`embeddedRecipes`, `RecipeEngines`, `BouncerRunSubdir`), `battencli` `childRecipeSupported` and its refusal text, the run-profile template and defaults, the seed test, the `boardcli`, `configsync` and `runprofile` tests.
   - Not covered: `docs/overview.md` still lists the batten child recipes without arras (finding 1).
   - Not covered: the boardengine/battencli raddle-only branches stay raddle-only.
     I treat that as intended, because D6 and the filing of an arras entry are a different part's work.

2. **Split perch keys (`split_*`)**
   - Covered: the `Config` fields, `ConfigSpec` categories (fans guard, models and judge none), the `reviewbudget` auto-accept guard, `ConfigOpenMaps`, `fanKeys`, `segmentModelKeys`, `Caps`, `template.yaml`, the `configreg` tests and the `loomcli` wiring maps.
   - Not covered: `docs/overview.md` loom.yaml key paragraphs name only the discussion and code keys (finding 2).
   - Not covered: `internal/configsync/configsync.go` (`formerLoomValues`) is not touched. It lists former template values of keys that already existed, so new keys need no entry.

3. **`Raddle-Review` segment and `PerchSplit`**
   - Covered: `loomPerches`, `loomSegmentPerches`, `loomSegmentSettings`, `ResolveJudge`, `ResolveReview`, `InterruptPolicies`, `envkit.FullEnv`, the `Raddle-Review` segment in the wiring tests.
   - Not covered: `retireSkipMergeReview` iterates `PerchDiscussion`, `PerchPlan` and `PerchCode`. It migrates an old key that never existed for split, so there is no gap.

4. **`loom-rubric-raddle-review` stencil**
   - Covered: the embed var, the `entries` registry and the `rubric_test` row.
   - Wording matches D3: the decision record with its `Added after Discussion` entries as authority, and each of the five judgements.
   - Wording matches D3: the proposal spec's mechanical checks are left to the gate.
   - The `{{.specs_dir}}/raddle/…` path matches the existing raddle stencil.
   - No finding.

5. **Arbiter description (`arbiterprompt.go`)**
   - Covered: the doc comment, the new branch, the `recipekeyed_test` row.
   - No finding.

6. **Package docs and `docs/overview.md`**
   - `loomrecipe`, `loomcli`, `shedrun` and `runprofile` doc.go and the glossary are updated.
   - `overview.md` lines 330, 335 and 1034 are updated.
   - Not covered: the batten `recipe` list (about line 1522) and the batten `Run-Shed` list (about line 1791) are not updated (finding 1).

7. **`PATTERN-refusal-way-forward`**
   - The diff adds no new refusal.
   - The existing raddle-approve perch refusal names a way forward.
   - No finding.

8. **`PATTERN-shed-recipe-registry`**
   - `arras` declares `contract: loom`, and no verb branches on the name.
   - The arbiter and perch-mode detection go by rows.
   - No finding.

9. **`PATTERN` declared changes and `Pattern-Departure` trailers**
   - The directive lists no declared changes, and neither commit carries a trailer.
   - No finding.

10. **Commit messages against the diff**
    - Every claim in the two bodies is borne out by the diff.
    - The summary's "no test was run in this session" is not a diff claim.
    - No finding.

**Findings**

1. **LOW, consistency** — `docs/overview.md` ~1522 and ~1791.
   - The batten `recipe` list ("one of `tapestry`, `motif`, `raddle`, `darn` and `sampler`") and the `Run-Shed` list ("tapestry, motif, raddle, darn or sampler") omit `arras`.
   - The diff makes `childRecipeSupported` admit `arras` (`internal/battencli/wire.go:607`) and adds it to the refusal text (`wire.go:613`).
   - The same diff updated `overview.md` at the lines for loomrecipe, shedcli and the `lym loom` verbs.
   - Evidence: the `overview.md` greps above show lines 1522 and 1791 unchanged.

2. **LOW, consistency** — `docs/overview.md` ~1318–1347 and ~1493–1497.
   - The loom.yaml key paragraphs name `discussion_*` and `code_*` keys only: reviewer and fixer fans, the fan-key list, the hard caps and `discussion_auto_accept`/`code_auto_accept`.
   - The diff adds `split_max_bounces`, `split_hard_max_bounces`, `split_auto_accept`, `split_reviewer_model`, `split_fixer_model`, `split_judge` and both `split_*_fan` keys, with the raddle-key fallback for `split_fixer_model`.
   - None is described in `overview.md`.
   - Evidence: `internal/loomengine/template.yaml` (diff lines 817–852) against `docs/overview.md:1322-1347` and `1493-1497`.

Tool calls made: none. My context already held every file I used.
