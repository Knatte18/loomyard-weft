```yaml
lens: fix-proposal
outcome: reported
```

No findings.

Lens: fix-proposal. I challenged each finding fixed as proposed (F1, F3, F4, F6, F8) against the code.

- **F1 (judge reads the charter):** The judge template says it never opens the artifacts, "never open the artifacts or the fixer's diff" (`bouncer-template-judge.md:22`, `:120`). The focus file and decision record the facts now name are not the artifacts it excludes, and the rubric explicitly tells the judge to read the charter, so there is no contradiction.
- **F4 (copies under harden-live):** `gatecli` accepts `--copies` with `--gate` and explicit targets for any one-tier key below tier3 (`internal/gatecli/run.go:397-401`, `:451`). The proposal holds.
- **F4, retired key:** Removing `harden-live-copies` from `goReadGates` and the template breaks no gateslot validation I could find. The `unknown`/`undeclared` hits in `config.go` and `tiers.go` concern `tier_parallel` entries, not gate keys. A hub whose `gate.yaml` still holds the old key may behave differently; I did not verify that.
- **F6 (fixer rules via target instructions):** Every other recipe's `BurlerFix` row with a `target` (arras, motif, tapestry) sets only `paths` and no `instructions`. Rendering the target's instructions into the fix work therefore changes no other recipe's fixer prompt.
- **F3, F8 (Preflight):** These follow from the code in context. Preflight refuses an absent `gate.yaml` before anything else, and reads the board entry only when `cfg.Packages` is empty.

Calls made beyond my context:
1. `grep` of the judge stencil and `gateslot` for artifact and unknown-key rules: I had never read the judge template and had no view of gate-key validation.
2. `awk` over the recipe YAMLs for `target:` blocks in `BurlerFix` rows: to see whether F6 changes other recipes.
3. `sed` of the arras, motif and tapestry fix profiles: to confirm they set no `instructions`.
