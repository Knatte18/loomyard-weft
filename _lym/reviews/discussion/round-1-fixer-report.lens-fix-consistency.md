```yaml
lens: fix-consistency
outcome: reported
```

I checked whether the round-1 changes to `_lym/discussion/decision-record.md` agree with the sections around them.

Sections checked against the changed text:
- **D1 bullets, Constraints and D1 acceptance (F1):** they agree. `board-scope` keeps its list and ends with the emphasis line. The Constraint names that rule, and the acceptance line says `TestConfigTemplate` passes unchanged. The D1 rationale is unaffected.
- **D5 sentence and D5 acceptance (F3):** they agree. Both name `webster_reviewer_fan` and `webster_fan`.
- **D6 heading, body, D6 acceptance, D7 first bullet and Auto-mode assumptions (F2 and F6):** these agree with each other. Goal, Scope and Open risk 1 do not (findings 1–3 below).
- **D7 second bullet (F4):** it agrees with D7's rationale. The Turn 5 line in the support log still names the old D7 sites. That log is a history, and the new Round 1 entry records the correction, so this is not a finding.
- **Open risk 2 (F5):** it agrees with the Scope Out line and with D5's load-failure bound.

Findings:

1. **LOW.** Scope, In, second bullet: "The `light` and `docs` profiles in `internal/runprofile/template.yaml`."
   D6 now also edits `speed`'s loom block, but Scope still names only `light` and `docs`.
   Evidence: D6 says "the `speed` profile's `loom` block … each set `loom.code_reviewer_fan: ""`".

2. **LOW.** Open risks, first bullet: "the operator re-sets `light` and `docs` with `lym config profiles --set` after the landing deploys."
   The F2 fix made `speed`'s loom block solo, but the operator step does not include `speed`, so the live hub's `speed` loom code review stays fanned.
   Evidence: the live `_board/_lym/config/profiles.yaml` has a `speed` `loom` block with `code_max_bounces: 1` and no fan key, so its code round runs over the hub's fan.

3. **LOW.** Goal, line 6: "a solo code review in the `light` and `docs` profiles".
   The Goal does not mention `speed`'s loom block, which D6 now covers. Auto-mode assumptions does list it as a pick the entry does not name, so a one-clause mention in the Goal would be enough.
   Evidence: the D6 heading "`light`, `docs` and `speed`'s loom block run a solo code review".

Tool calls:
- I read the live hub `_board/_lym/config/profiles.yaml` to see whether its `speed` block runs a code round. My context held only the template, not the live file.
