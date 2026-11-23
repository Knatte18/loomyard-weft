```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the stale-wording searches for F1, F2 and F4. I found no stale sites.

No findings.

- F1 and F2 (the "cannot judge the tests hold the run" wording): every remaining hit already says a non-go hub passes. The hits are `contracts/stencils/darn/darn-template-write.md:111`, `contracts/recipes/darn-recipe.yaml:137`, `internal/shedrecipe/doc.go:65`, `internal/gatecli/redfirst.go:73` and `docs/overview.md:1114`. No other copy of the old hold wording is left outside `_lym/reviews`.
- F4 (the exit-5 sites): the only non-test production uses of `gateslot.RedFirstUnjudgedExit` are `internal/loomshed/redfirstgate.go:41,216` and its definition at `internal/gateslot/redfirst.go:14-17`. The `exit 5` hit in `internal/creelcli/cli.go:387` is an unrelated verb.
- F3 (the runner-crash test row): I did not search this one. It rests on the single `!res.RunnerCrash` conjunct at `internal/loomshed/redfirstgate.go:216`, which your own grep already covered.
- I did not check whether other copies of the summary's "still holds the run as before" claim exist. Only `_lym/landing/summary.md` was in scope.

Tool calls (3 of 6):
1. grep for "cannot judge the tests" variants across the repo. Your context held only the files you read, not every copy of that wording.
2. grep for `RedFirstUnjudgedExit`, "exit 5" and "unjudged" outside tests. The set of exit-5 sites was not in your context.
3. grep for "scoper is not go" and "non-go" in contracts, docs and `doc.go` files, to find any twin of the new passes-on-non-go sentence.
