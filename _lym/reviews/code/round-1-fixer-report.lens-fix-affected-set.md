```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: rerun the set searches behind F1, F3, F7 and F10 and check the fix covers every member.

**Findings**

- **LOW** — `internal/testcover/linemutant.go:31` (F7, same rule restated; not changed by the fix). The comment still reads "an if statement with no init statement whose every branch only logs". The fix to `logsOnly` also requires no call in the condition, and `internal/testcover/doc.go:75-76` and the `logsOnly` doc comment now say so. This file's comment was left stale. Evidence: the grep for "only logs" lists it next to the updated `doc.go`.

- **LOW** — F1, set of language-literal sites. The new `mutationStep.run` case goes through `HasGoCoverageMaps`, so the added instance is gone. The review allowed, but did not require, moving the existing sites in the same change. These sites still branch on the literal for the same "has Go coverage maps" meaning: `internal/gatecli/reach.go:84`, `internal/gatecli/redundancy.go:109`, `internal/gatecli/mutate.go:151` and `internal/gatecli/reachverb.go:149`. The `Scoper` grep finds them. The fixer should record this as a deliberate choice in the fixer-report, not leave it silent.

**Searches that found nothing missed**

- F3: every mention of `mutation_max_mutants` and `mutation_budget_min` covers the mutation report step too. The template comments (`internal/gateslot/template.yaml:11-12`), `config.go` and `gateslot/doc.go:205-207` all do. `internal/gatecli/doc.go:763-767` already states the even spread, and `docs/overview.md:982-983` is not wrong.
- F10: a grep for "judged across" and "every package and file" finds no remaining occurrence outside `_lym`, apart from the research report.

**Tool calls**

1. A grep for the `mutation_max_mutants` and `mutation_budget_min` mentions, and for the "only logs", "judged across", "every package and file" and "spread over the changed" phrases. Context held no repo-wide hit list for these.
2. A grep for the `Scoper` and `scoperGo` literal sites. Context held only the earlier partial grep.
3. A read of `internal/gatecli/doc.go` lines 750-770, to check the mutation-verb wording on the cap spread against the `mutate.go` docs.
