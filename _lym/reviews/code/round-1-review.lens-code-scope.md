```yaml
lens: code-scope
outcome: reported
```

Task: trace the diff's changes to the decisions in the decision record (code-scope lens).

**Decisions**

- G1: the template default for `reviewer` and `fixer` is `sonnet[high]` alone, with no rotation.
- G2: which models a campaign uses, a rotation included, is the operator's choice per run, set in `run_settings` under `hardener`.
- G3: the template comments, and any doc or pattern text that describes the rotation as the default, say so.
- G4: check that `run_settings` takes `hardener.reviewer` and `hardener.fixer` as lists, and fix it if not.
- A1 (added after Discussion, parent, 2026-11-23): preflight accepts a one-entry reviewer list, and still refuses neighbouring equal entries in lists of two or more.

**Per change**

- C1 `internal/hardenshed/template.yaml:5-6`, values changed to `["sonnet[high]"]`: traces to G1.
- C2 `internal/hardenshed/template.yaml:5-6`, comments describing the default and the per-run rotation: traces to G3 and G2.
- C3 `internal/hardenshed/doc.go:13-14`, Config section stating the default and the per-run choice: traces to G3 and G2.
- C4 `internal/hardenshed/doc.go:96-97`, Method section no longer says rounds rotate models: traces to G3.
- C5 `internal/hardenshed/config_test.go:53`, `TestLoadConfig` defaults: forced by C1.
- C6 `internal/configreg/runsettings_test.go:93-100`, new list row: traces to G4. No code fix was needed, since `PrepareRunSettings` marshals list values as they are (`internal/configreg/runsettings.go:69`).
- C7 `internal/hardenshed/campaign.go:48,120-126`, one-entry list passes, and the doc comment names only lists of two or more: traces to A1.
- C8 `internal/hardenshed/campaign_test.go:106-108`, `ReviewerListOfOneEntryPasses` row: forced by C7.

**Per decision**

- G1: built by C1, with C5 forced by it.
- G2: built by C2 and C3. The diff changes no mechanism, because `run_settings` already takes the keys.
- G3: built by C2, C3 and C4. It is only partly built: see F1.
- G4: built by C6.
- A1: built by C7 and C8.

No change lacks a decision, so no unrequested scope.

**Findings**

- F1. BLOCKING, class `scope`, at `contracts/stencils/harden/harden-rubric-review.md:98`. The judge rules CONVERGED only on a round "whose reviewer model differs from the previous round's". That rule assumes rotation is the default. Under G1's default every round's reviewer is `sonnet[high]`, so a campaign run on the default can never converge.
  - The diff leaves this stencil untouched, so G1 and G3 are not fully built.
  - G3 asks that text resting on rotation as the default be brought in line, and the default must work as a single-model campaign. A1's rationale says so too.
  - Evidence:
    - `contracts/stencils/harden/harden-rubric-review.md:98` holds the rule.
    - `internal/hardenshed/campaign_rows.go:186` writes each round's reviewer model into the facts the judge reads.
    - `internal/hardenshed/template.yaml:5` sets the one-entry default.
  - Fix: make the convergence condition require a model change only when the round's reviewer list holds more than one model, or drop the condition. Keep it consistent with `doc.go`'s Method section.
  - This overlaps the affected-set lens.

Tool calls: none. Every item came from the inherited context.
