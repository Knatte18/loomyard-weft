```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens for the review-setup-tuning diff. For each change I list the set it forces, and whether the diff covers it.

**Per change, set found, covered or not**

1. `code_reviewer_fan` goes from `CategoryGuard` to `CategoryNone` in `internal/loomengine/config.go`. Covered.
   - The `ConfigSpec` doc comment now names the exception (`config.go:501`).
   - `docs/overview.md:584` names it as open and the other five as closed.
   - `pattern/PATTERN-config-layers.md` carries the Guard exception.
   - The `internal/configreg/configreg_test.go` guard list drops the key.
   - Tests cover the local-layer admit, the retired `darn_reviewer_fan`, `review_fan`, `webster_fan` and `webster_reviewer_fan` routes, and the five remaining guards.
   - `internal/configsync/configsync_test.go` covers the sparse local layer.
   - `ConfigOpenMaps` (`config.go:485-496`) is about shape, not category, so it needs no change.
   - The `config.go:22` comment ("all three of these keys are closed") refers to `discussion_advisors`, `discussion_lenses` and `plan_lenses`, so it is still right.
   - The `internal/loomengine/template.yaml:20` comment makes no claim about layers.
   - The recipe files (`contracts/recipes/*`) and `internal/shedrecipe` only read the setting by key, so they are unaffected.
   - The `internal/configcli/doc.go:56` mention is about value shape, not category, so it is still right.
   - `internal/battencli/runsettings_test.go` is updated for the profile block's new key.
2. The profile template changes (`light`, `speed` loom, `docs` darn get `code_reviewer_fan: ""`). Covered.
   - `internal/runprofile/doc.go` is updated.
   - `TestTemplateLadder` and `TestTemplateDocsProfile` assert the new keys.
   - The `battencli` expected output is updated.
3. The lens removals and renames (`goal-scope`, `board-fit` and `code-pattern-fit` out; `board-scope` and `repo-invariants` in). Covered.
   - A repo-wide grep outside `_lym`, `.lym` and `docs/research` finds no remaining reference to the removed lenses except the retired-lens assertion in `config_test.go:444`. The `docs/research` hits are measurement reports, which the prose rule exempts.
   - `docs/dotnet-hub.md` and `internal/lenses/sets/csharp.yaml` drop `code-pattern-fit`.
   - Tests using `goal-scope` as a sample lens name (`internal/loomengine/discussiontable_test.go`, `internal/lenses/config_test.go` `TestLoad`, `TestCheckLoad`) are moved to `board-scope` or `interactions`.
   - `internal/configsync/configsync_test.go` truncates the `twill-discussion` fan to `[:5]`, which is its old template state with the one-lens-shorter fan.
   - No file under `contracts/stencils` or `contracts/specs` names a changed lens (grep found none), so Stencil wording has nothing to judge.
   - `loom-rubric-code-review.md`'s "Test economy" bullet is unchanged, as D4 requires.
4. The `way-forward` emphasis line, the `code-affected-set` prose and the narrowed `code-tests` prose. Covered by the `config_test.go` sentence pins and the emphasis helper.
5. The template header sentence that exempts procedure lenses now names `board-scope`. Covered by the header edit and the `procedureLenses` and `baseProcedureLenses` seeds in `config_test.go`.

**The repository's own rules:** the docs-in-the-same-commit rule holds for D7 (`doc.go`, `docs/overview.md`, `docs/dotnet-hub.md`, `config.go` comments, the `pattern/` background). The `PATTERN.md` overview line stays unchanged, as the record says.

**Findings:** none.

**Tool calls**
1. `grep reviewer_fan` over Go, md and yaml outside tests: the manifest held only the diff, not the callers, recipes or other comments.
2. `grep` for the lens names outside `template.yaml` and `config_test.go`: to find stencils, specs or docs that name a changed lens.
3. `sed` of `config.go:8`, `config.go:20-23`, `config.go:485-496`, `internal/configcli/doc.go:56` and `internal/loomengine/template.yaml:20`: to check that the surrounding comments are not stale.
4. `grep` for "closed to the local layer" and "all six" in docs and comments: to catch any remaining "all fan keys closed" claim.
