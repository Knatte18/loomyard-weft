```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every claim and bound in the record.

**Claims attacked:**

1. Constraint "procedure lenses, those of the `review-*` and `fix-*` fans and `board-scope`, end in their own list, every other lens ends with the emphasis line", with D1's acceptance line "still fails any other lens of a non-procedure fan": **broken by D2's `way-forward`.**
2. D5 bound "a value can name only a fan the guarded `lenses.yaml` defines, or `""`", read with "an agent can thin its own run's code review to solo": **broken.** The bound understates what the key lets through.
3. D5 "loom's retired `webster_reviewer_fan` and `webster_fan` … admitted with it", with D5's acceptance line "a local layer … holding … loom's retired `webster_reviewer_fan` … is admitted": **broken for the hand-edited local-file route.**
4. D5 "a wrong name fails the pair's loom config load through `ValidateFanKeys`, whose refusal names the key and the fix": held (`internal/lenses/fankeys.go:76-77`).
5. D5 "a task's `run_settings` reach a pair only at Seed-Child": held. `PrepareRunSettings` is reached only through `LayerRunSettings`, the battenshed Seed-Child deps and `checkProfiles` (`internal/configreg/runsettings.go:105-154`, `internal/battenshed/deps.go:360`).
6. D5 "`code_max_bounces: 0` … stays the stronger switch": held. The budget keys are `CategoryNone` through reviewbudget (`internal/loomengine/config.go:498`).
7. D5 "the retired darn keys … `configreg`'s key move carries to `loom.code_reviewer_fan`": held (`internal/configreg/keymove.go:30,32`, and `moveRunSettings` runs before the check at `runsettings.go:46`).
8. D5 rationale "configreg checks a profile block … through `checkLocalDocument`'s `ValidateLocal`, which refuses a guard key": held (`runsettings.go:135-143`, `internal/configengine/layers.go:291-293`).
9. D6 "`solid` and `wide` … take the template's `["review-code", ""]` where no layer sets the key": held (`internal/loomengine/template.yaml:20`).
10. Constraint "D6's value survives reconcile": held. Round 1 confirmed it, and nothing in my context contradicts it.
11. Open risk 2 "a fan one of the six fan keys names fails every loom config load": held. `ValidateFanKeys` runs `lenses.Load` and then `ResolveFan` per entry (`fankeys.go:70-77`).
12. Constraint "`profiles.yaml` is seed-only": held (`internal/runprofile/template.yaml:8`).

**Findings:**

- **MEDIUM · design** · `decision-record.md` D2, the Constraints lens-prose line and D1's acceptance line.
  - **Summary:** after D2 drops `way-forward` from `review-plan`, the lens is in no `review-*` or `fix-*` fan, only in `twill-plan`. The record's own rule then requires it to end with the emphasis line, but its prose ends "Your list: per refusal, the next step, progresses or not."
  - **Effect:** `TestConfigTemplate` computes the exemption from the `review-*` and `fix-*` fans in `wantOrders`, so it will fail on `way-forward`. D2 says only "`way-forward` stays defined and stays in `twill-plan`", and no decision says whether its prose gains the emphasis line or the exemption widens to name it, as D1 does for `board-scope`.
  - **Evidence:** `internal/lenses/template.yaml:288-293` (the `way-forward` prose), `:398-414` (fans), `internal/lenses/config_test.go:296-300` (exemption from `review-`/`fix-` fans), `:341-343` (the emphasis-line check).
- **MEDIUM · design** · `decision-record.md` D5, the first and third Bound bullets.
  - **Summary:** the agent's power is stated as "thin … to solo", but a pair's local layer can re-point `code_reviewer_fan` at any fan `lenses.yaml` defines.
  - **Example:** `standard`, `plan`, `twill-discussion` or `fix-code` all pass. The Code-Review fan would then run discussion or plan lenses, or fixer prose, over a diff.
  - **Effect:** the record states no bound on that substitution beyond "the rubric and whole-diff read are unchanged". This is the attack-surface question's "what it can now let through".
  - **Evidence:** `ValidateFanKeys` checks only that the name resolves (`internal/lenses/fankeys.go:65-78`), and `template.yaml:362-435` defines every one of those fans.
- **LOW · consistency** · `decision-record.md` D5's sentence on retired loom keys and its D5 acceptance line.
  - **Summary:** `webster_reviewer_fan` and `webster_fan` are renamed before the check only on the run_settings and profile route. A hand-edited local file holding `webster_reviewer_fan` is checked raw by `Load`'s local-layer `ValidateLocal`, with no Migrate, and is refused as "not in the loom template" until a reconcile migrates the file.
  - **Effect:** "admitted with it … the same bound holds for every one of these routes" overstates the local-file route. The acceptance line's "a local layer … holding … loom's retired `webster_reviewer_fan` … moved onto it" holds only after reconcile.
  - **Evidence:** `internal/configengine/layers.go:476-481` (`ValidateLocal` on raw local data), `:275-277` (not-in-template refusal), `internal/configreg/runsettings.go:73-84` (Migrate only on that route).

**Tool calls:**

1. `grep` for `ValidateFanKeys` in `internal/lenses/fankeys.go`: my context lacked the fan-name check that D5's bound and Open risk 2 rest on.
2. `sed` of `internal/configengine/layers.go:460-495`: my context lacked how `Load` checks a raw local file, needed for claim 3.
3. `grep` for the `PrepareRunSettings` and `LayerRunSettings` callers: my context lacked the callers that D5's "only at Seed-Child" claim rests on.
4. `grep` for `code_reviewer_fan` in `internal/loomengine/template.yaml`: my context lacked the template default that D6 cites.
