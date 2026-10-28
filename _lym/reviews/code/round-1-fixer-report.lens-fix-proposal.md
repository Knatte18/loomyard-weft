```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F6) through the fix-proposal lens.

no findings

I built a counter-case for each proposal and none held:

- **F5, trimming `dirtyWayForward` off the review cause.** `clean()` always builds its reason through `dirtyReason`, so the suffix is always present and `TrimSuffix` always applies. Other `dirtyReason` consumers still get the same text, because the constant keeps the original wording. The halt text still joins cleanly: `...a/a.go; session log: ...`.
- **F4, comparing the review's `ModelChoice` to the fixer's.** Both tables derive from `cfg.Fixer`, so the check would catch a review wired to another config key. It cannot pass vacuously: `claude:opus` is the escape form, and `Resolve` returns it with `Engine` and `Model` set, so the choice is non-zero.
- **F4, the timeout assertion.** It compares against `FixerTimeoutMin` times a minute, with the fixture set to 5, so a zero timeout would fail it.
- **F3, the exact session count.** `wantSessions` is 1 only for the over-cap row that scripts a review. The failing-gate and within-caps rows spawn none, so no row's expectation is wrong.
- **F2, the timeout, re-prompt and start-failure rows.** The `reviewTimeout` and `reviewStartErr` fake fields apply to the fix review role alone, so the fixer session in the same run is unaffected. The re-prompt row's two scripted turns match `fixReviewReprompts = 1`.
- **F1, the approval record and the moved HEAD.** The fixer read the record through a refused push, because a landing removes it. That avoids the review's literal proposal, which would have found no record. The seeded `Reviewed: "e9"` against HEAD `f1` fails `rec.Reviewed == head`, so it forces a second review.
- **F6, the two doc edits.** They match the already-updated `config.go` and `template.yaml` comments.

Calls made:
1. `grep` of `Registry.Resolve` in `internal/modelspec/registry.go`, to check that `claude:opus` resolves to a non-empty `ModelChoice`, which my context lacked.
