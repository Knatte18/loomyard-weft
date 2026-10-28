```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens over `decision-record.md` and `support-log.md`. I tried to break each claim and bound the record states.

**Claims attacked**

- D1, "at most six tool calls" (3 fixed reads plus up to 3 more): **held**.
- D1 with the Constraint "procedure lenses end in their own list": **broken**, see F1.
- D2, the writer-covers-reviewer test gains `repo-invariants` as its one reviewer-only lens: **held**. The `absorbedBy` check in `internal/lenses/config_test.go:312–318` is the only place that needs the exemption.
- D3, `code-pattern-fit` named "A broken invariant" and "A pattern the parent changed", and the record drops them: **held** (`internal/lenses/template.yaml:317–326`).
- D5, a wrong fan name fails the pair's loom load through `ValidateFanKeys`, and the refusal names the key and the fix: **held**. The fix text is in `internal/lenses/fankeys.go`. `LoadConfig` runs at `internal/loomcli/wiring.go:378`, and darn and pawl runs are wired through loomcli too.
- D5, the retired darn keys `darn_reviewer_fan` and `review_fan` are admitted with the key: **held**, but the list is incomplete, see F3.
- D5, the five other fan keys stay refused, including the retired `darn_fixer_fan` and `review_fix_fan`: **held** (`internal/configreg/keymove.go`).
- D5, `code_max_bounces: 0` from the same local file is the stronger switch: **held**. `reviewbudget` budgets are no guard (`internal/loomengine/config.go` `ConfigSpec` doc).
- D5, `run_settings` reach a pair only at Seed-Child: **held**. In `contracts/recipes/batten-recipe.yaml:41–86`, Seed-Child runs once before Run-Shed, and Run-Shed's `on_stuck` re-enters Run-Shed, never Seed-Child. A `run_settings` change on a run-held entry is allowed (`internal/boardengine/doc.go:83`) but reaches no running pair.
- D5, "A value can name only a fan the guarded `lenses.yaml` defines": **held as far as the existing guard reaches**. `lenses` is a per-worktree module (`internal/configreg/configreg.go:134`), so its tracked worktree layer is as open as before. D5 adds no reach there.
- D5 rationale, a profile block is refused through `checkLocalDocument` → `ValidateLocal`: **held** (`internal/configreg/runsettings.go` `checkProfiles` → `PrepareRunSettings`).
- D6, `speed`, `solid` and `wide` "take the template's `["review-code", ""]`": **broken in general**, see F4.
- D6, every block passes configreg's check after the reclassification: **held**.
- D6, a `light`-seeded pair loads one empty entry and runs a solo round: **held**. `FanList` loads `""` as `[""]`, `ValidateFanKeys` skips the empty entry, and `fanSetting` returns a nil list, which means solo (`internal/shedrecipe/entries_settings.go:84–87`).
- Constraint, D6's value survives reconcile: **held**. `ReconcileAll` passes a nil strip for local files (`internal/configsync/configsync.go:424`). `Prune` keeps the key because the template has it.
- Constraint, the sparse lenses reconcile lets template edits reach untuned hubs: **held**.
- Constraint, `profiles.yaml` is seed-only: **held** (`internal/runprofile/doc.go`).
- Constraint, no file under `contracts/stencils` names a changed lens: **held** (grep found none outside `internal/lenses` and tests).
- Open risk 1, live `profiles.yaml` has no `pawl` blocks and `docs` sets `code_max_bounces: 0`: **held** (`_board/_lym/config/profiles.yaml`).
- Open risk 2, a hub fan naming a deleted lens "fails every loom config load": **broken as stated**, see F2.

**Findings**

- **F1: MEDIUM, design, D1 and Constraints ("procedure lenses end in their own list").**
  - Under D1 the emphasis-line test forces `board-scope`'s prose to end with "Report anything else you notice too — emphasis, never exclusion.", which contradicts D1's "Ends in its list" and "Reports only a contradiction".
  - Why: the exemption from the emphasis line is computed only from lenses in `review-*` and `fix-*` fans (`internal/lenses/config_test.go:293–297, 336–342`). `goal-scope` and `board-fit` are exempt today only because they sit in `review-discussion`. D1 removes the pair from `review-discussion` and puts `board-scope` only in `twill-discussion` and `discussion`, so the exemption no longer covers it. `leanness`, which is only in a twill fan, ends with the emphasis line for the same reason (`template.yaml:132–137`).
  - The record and its acceptance criteria say nothing about this: either the prose changes or the test's exemption rule changes.
  - Evidence: `internal/lenses/config_test.go:293–297, 340–342`; `internal/lenses/template.yaml:33–36`.
- **F2: LOW, consistency, Open risks bullet 2.**
  - The claim that a hub fan naming `goal-scope`, `board-fit` or `code-pattern-fit` "fails every loom config load" holds only for a fan one of the six reviewer or fixer fan keys names.
  - `LoadConfig` validates only `fanKeys(cfg)` (`internal/loomengine/config.go:1225`, `fanKeys` lists six keys). `discussion_lenses` and `plan_lenses` are not validated at load.
  - Counterexample: a hub that tuned `twill-discussion` to include `goal-scope` loads fine, then fails later when the Discussion-Write lead resolves its fan. That is a different failure point from the one the risk names.
  - Evidence: `internal/loomengine/config.go:403–409, 1225`.
- **F3: LOW, design, D5 bound (retired keys admitted).**
  - D5 names only the darn key moves, but loom's own `MigrateConfig` also renames the retired `webster_reviewer_fan` to `code_reviewer_fan` (`retiredPerchKeyRenames`). `PrepareRunSettings` runs `module.Migrate` before `checkLocalDocument`, so a profile block or `run_settings` holding `webster_reviewer_fan` (or `webster_fan`, through `webster_reviewer_fan`) is also admitted after D5.
  - The record states no bound for these routes, and acceptance criterion D5 does not cover them.
  - Evidence: `internal/loomengine/config.go:1023, 1035–1040`; `internal/configreg/runsettings.go:72–85`.
- **F4: NIT, consistency, D6 ("`speed`, `solid` and `wide` set no fan, so they take the template's `["review-code", ""]`").**
  - They take whatever the hub's loom stack resolves, which is the template value only when no layer sets `code_reviewer_fan`. On this hub `loom.yaml` sets only `discussion_reviewer_fan`, so the claim holds here but not in general.
  - Evidence: `_board/_lym/config/loom.yaml:1`.

**Tool calls**

1. `configreg.go` module rows for `lenses` and `loom`, plus a Seed-Child re-entry grep: to check whether `lenses` has a worktree layer an agent can write, and whether Seed-Child reruns. The manifest lacked the module registration flags.
2. `boardengine/doc.go` `run_settings` lock lines and the `batten-recipe.yaml` listing: to test D5's "`run_settings` reach a pair only at Seed-Child". The manifest lacked the batten recipe.
3. `batten-recipe.yaml` row and route lines: to confirm that no route re-enters Seed-Child. The previous call's output was cut short.
4. `loomengine/config.go` grep for `ValidateFanKeys` and the `discussion_lenses`/`plan_lenses` fields: to test Open risk 2's "every loom config load". The manifest lacked the `LoadConfig` validation site.
