```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, listing what each changed signature, output or behaviour forces outside the touched lines.

**Per change: set found, covered or not**

- **`runDeploy` → `RunDeploy`, `deployTimeout` → `DeployTimeout`:**
  - Set: a repo-wide grep of the old names returns nothing. The `battencli/land.go` closure and the `battenshed` land code and test were updated.
  - Covered.
- **`Fabric.CommitExists`:**
  - Set: the only caller is `internal/adoptcli/deploy.go`. A search of `fabricengine` for other users of `SHAExists` found none.
  - Covered.
- **The `deploy` envelope field and the help text:**
  - Set: the grep for `prime_beam_steps`/`prime_beam_warning` outside `adoptcli` returns nothing, so no other doc or stencil lists envelope fields. `docs/overview.md:447-451` describes the adopt steps (reconcile, stops, holds, owner write, checks) and does not mention the deploy step. That is a prose summary of the verb, not a field list.
  - Not covered: see finding F-1.
- **New `DeployState` and `RunDeploy` fields on `adoptDeps`:**
  - Set: `adoptDeps{` is constructed in `deps.go` and `adopt_test.go` only. Both are updated, and a nil `DeployState` is handled in `deployStale`.
  - Covered.
- **New `adoptcli` imports of `battenshed` and `buildvcs`:**
  - Set: `pattern/PATTERN-cli-cobra.md:36` enumerates what `adoptcli` calls: `fabricengine, pairteardown, boardengine, tenderengine, tendercli, battencli, landingshed`. `battenshed` and `buildvcs` are not in the list.
  - Not covered: see finding F-2.
- **Declared PATTERN changes:** the directive's constraints section declares none, and the diff touches no `PATTERN.md` or `pattern/` file. Nothing to check.
- **Commit message and departures:** the single commit carries no `Pattern-Departure:` trailer. I found no departure that needs one.
- **Stencil wording:** the diff changes no stencil.

**Findings**

- **F-1: LOW, consistency.** `docs/overview.md:447-451` (the adopt bullet) does not name the new deploy step. It describes the verb's steps in order and now omits one that can refuse the verb. The repository's own rules say docs a change touches land in the same diff, and this doc names `internal/adoptcli` as the package. Evidence: `docs/overview.md:448`; the diff has no `docs/` change. Fix: add the deploy step between the hold and the owner write in that bullet.
- **F-2: LOW, consistency.** `pattern/PATTERN-cli-cobra.md:36`, the `adoptcli` deviation line, enumerates the packages adopt calls. The diff adds `battenshed` (for `DeployTimeout`) and `buildvcs`, and neither is listed. `battencli` and `landingshed` were already listed. Evidence: `pattern/PATTERN-cli-cobra.md:36`; imports at `internal/adoptcli/deploy.go:14-20`. Fix: add `battenshed` and `buildvcs` to the list, or reword it so it no longer enumerates.

**Tool calls**
1. A grep for the old names, the envelope field names, the `adoptDeps{` constructors and the `battencli`/`battenshed` mentions in the PATTERN docs. My context lacked the repo-wide sets.
2. A grep for docs naming `lym adopt`, a 2000-byte help-cap test and `CommitExists`/`SHAExists` users. My context lacked these.
3. A read of `PATTERN-cli-cobra.md` lines 20-60, plus the `docs/overview.md` mentions of adopt. I needed the exact deviation text.
