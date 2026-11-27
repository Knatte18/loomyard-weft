```yaml
lens: fix-consistency
outcome: reported
```

Task: consistency check of the fix change set against its surroundings. Three findings.

1. **MEDIUM.** Changed text: `internal/gatecli/only.go#markedScope`, the new "would run no test" refusal.
   - The way forward says "set `gate.tiers.tier0.dir` to the module that holds the marked tests", a config change, with no `lym config gate` route and no `roleClause`.
   - The sibling refusal added beside it (the unlisted-tier one) uses `configRefusal`. `gatecli/doc.go` says of config-fix refusals that they close with the clause for a session that is not the prime, which takes none of the steps (`tiers.go#roleClause`).
   - The board-tidy agent runs this call, so the refusal hands it a step without saying who may take it. Pointing `tier0.dir` elsewhere would also re-scope every other gate key's tier0 run.

2. **LOW.** Changed text: `markedScope`'s doc comment ("a pass always means every marked test ran"), and `gatecli/doc.go` "So does a call that would run no test".
   - `run.go` returns before `withOnlyTests` and `computeScope` when `len(cfg.Tiers) == 0`. A `--gate upkeep` call on a hub with no tier runs only its lints and exits 0, so neither statement holds there.
   - The template says of that case "While gate.tiers is empty, every key runs with nothing to run", which now sits beside the new template sentence "a key no test is marked for, refuse[s] the call".

3. **LOW.** Changed text: the F4 behaviour (no upkeep call under a scoper other than `go`) and the F1 refusal wording.
   - `internal/gateslot/template.yaml:18` still describes `upkeep` as "(scope marked, run by the board-tidy agent on the main branch)" with no `go`-scoper condition, although the template's own default scoper is `none`.
   - Template line 20 says "a marked test of a tier the key does not list" refuses the call. The code refuses only a tier that no listed tier **is or covers**, as `gatecli/doc.go` states, so a test of a covered, unlisted tier runs.

I found no other stale twin in the stencils, the `dobbyengine` comments, the PATTERN files or the change description.

Tool calls: 1 call, a `git grep` for other descriptions of the marked scope and upkeep run plus a read of `internal/gateslot/tiers.go:124-138`; my context held neither the repo-wide matches nor the `ScopeMarked` constant comments.
