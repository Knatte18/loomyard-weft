```yaml
lens: code-scope
outcome: reported
```

**Per change**

- C1 — `internal/gatecli/only.go` `markedScope` drops the unlisted-tier refusal and removes `markedNames` (diff 001, lines 23–65): traces to G2's "derive the tiers from the marked tests" option; each marked test now runs under its own file's tier through the guard step, so no hub needs the key.
- C2 — `internal/gatecli/scopes_integration_test.go` rewritten case expecting the tier1 marked test to run instead of the refusal (diff 001, lines 74–91): forced by C1, since the refusal the old row asserted is gone.
- C3 — `internal/gatecli/doc.go` (diff 001, lines 5–16), the `markedScope` doc comment (diff 001, lines 23–30) and `internal/gateslot/template.yaml` comment (diff 002, line 9→10): forced by C1, as the old text described the removed refusal and the template advised `{tiers: [0, 1], scope: marked}`.
- `cli.go` round trip across commits 1311d492f and c368b94ac: net-empty in the range diff, so it maps to nothing and is no scope finding (no `cli.go` change lands).

**Per decision**

The answer key is the decision record alone; it holds no `Added after Discussion` entries.

- G1 (`## Fix`: set this hub's `gate.gates.upkeep` to `{tiers: [0, 1], scope: marked}` and do the hand check on main) — no change builds it, and the entry's own `## Task` section keeps "Setting this hub's key and the hand check on main" with the orch (decision-record.md:24). No scope finding.
- G2 (`## Task`: change the shipped template default to the marked tests' tiers, OR derive the tiers from the marked tests so no hub needs the key by hand) — built by C1 (with C2, C3 forced). The entry offers the template default as an alternative, not an additional requirement, so leaving the template value at `[0]` meets G2 (template.yaml:10, decision-record.md:23).

**Findings**

None.

**Tool calls**

None; the inherited context held everything this lens needed.
