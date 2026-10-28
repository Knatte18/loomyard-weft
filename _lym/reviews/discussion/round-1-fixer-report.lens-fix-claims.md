```yaml
lens: fix-claims
outcome: reported
```

fix-claims check: I tested every bound, behaviour and code fact that the fix added or rewrote against the code at HEAD.

No findings.

What I checked:
- **F1, `board-scope`:** the claim holds. `board-scope` sits only in `twill-discussion` and the dormant `discussion` fan, so it is in no `review-*` or `fix-*` fan. The template header (`internal/lenses/template.yaml:33-36`) and `TestConfigTemplate` (`config_test.go:296-300`, `:341-343`) hold such a lens to the emphasis line. `leanness` ends with that line, so the claim that the check "passes unchanged" holds once the lens prose ends with it.
- **F2, `speed`:** `speed`'s `darn` and `pawl` blocks set `code_max_bounces: 0` (`internal/runprofile/template.yaml:19-21`). The overview says 0 runs no review round, so "runs no code round" holds.
- **F3, retired keys:**
  - `PrepareRunSettings` runs `module.Migrate` before `checkLocalDocument` (`internal/configreg/runsettings.go`).
  - loom's `Migrate` is `loomengine.MigrateConfig` (`internal/configreg/configreg.go:136`).
  - `webster_fan` becomes `webster_reviewer_fan` (`config.go:1022`) and then `code_reviewer_fan` (`config.go:1035`).

  So "admitted under the same bound, renamed before the check" holds.
- **F5, where a deleted lens fails:** `fanKeys` lists only the six reviewer and fixer keys (`config.go:624-633`). `DiscussionLenses` and `PlanLenses` are resolved only when the writer lead builds its prompt (`discussiontable.go:101-104`, `plan.go:92-95`). So the late failure at Discussion-Write or Plan-Write holds.
- **F6, hub value:** "Take the hub's `code_reviewer_fan`, the template's value where no layer sets it" restates the reviewer's premise and is consistent with the stacked load.

Tool calls:
1. One grep for loom's `Migrate` wiring, the `DiscussionLenses`/`PlanLenses` callers and `config.go:1015-1028`. My context lacked the module registration, the writer-lead resolution sites and the `webster_fan` rename row.
