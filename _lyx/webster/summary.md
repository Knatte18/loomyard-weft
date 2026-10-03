# Review convergence: finding classes, a facts-driven judge and circling resolution

All 11 batches reported `done`, and the integration suite reported `status: OK` at `bfab99c6a`.

- **01 finding-class:** every burler review finding carries a required `class`; `design` is the gating class.
- **02 rubric-class-mapping:** each loom rubric defines `design` for its own segment.
- **03 judge-model-key:** Bouncer rows resolve their model from a new `judge` key.
- **04 verdict-vocabulary:** the judge verdict is `CONVERGED`, `CONTINUE` or `CIRCLING`; legacy words are read only at Call entry.
- **05 round-facts-file:** Go renders a per-round facts file from the reviews and ledgers.
- **06 judge-reads-facts:** the judge reads the facts, the latest review and the previous ledger.
- **07 circling-decision-record:** the operator's accept or continue is a recorded file with its own refusals.
- **08 bouncer-circling-resolution:** the Bouncer acts on the recorded decision.
- **09 bouncer-env-slug-budget:** loom tells every Bouncer row its slug and live bounce budget.
- **10 circling-verbs:** `lyx loom circling accept|continue` record the decision.
- **11 retire-classification-draft:** the roadmap entry and draft design doc are removed.

One deviation was reported, in batch 10: the cobra-tree test sits in `internal/loomcli/circling_integration_test.go` rather than `circling_test.go`, because the Test Tier Purity Invariant bars `hubforge.NewHub` from untagged files.
