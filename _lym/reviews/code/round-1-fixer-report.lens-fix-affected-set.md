```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the sets behind F1, F3/F4 and F5 (sites of the role clause, `namesProject` callers, plan-time notices) for members the fix missed.

Findings:

1. LOW — `internal/gatecli/tiers.go:119-139` (the F5 fix). The fix moved only the run-whole notice to after the last refusal. Two other plan-time notices still print before later refusals.
   - These are the "scope selected no test class, so tierN runs whole" line (`tiers.go:120`) and the `-skip` watch line (`tiers.go:138`).
   - A call that prints either and then hits the guard or vet refusals (`tiers.go:157`, `:165-170`) shows a stale notice before the error, which is the F5 symptom.
   - The review named only the run-whole loop, and the fixer-report should say the other two were left.
   - The `-skip` line is also printed by the later guard refusal path, since `request.guards` can still refuse after it.

2. No finding on F1. The set is the refusals that should close with `roleClause`.
   - `tiers.go:144` now ends with `roleClause`, matching `configRefusal` and the filter refusal at `:415`.
   - The unsuffixed refusals are `:91`/`:98`/`:103`/`:263`, which wrap a `cfg.Tier` error; `:165`, which is deliberately "drop --guards"; and `:426`, which is a value error.
   - None of those is in the doc.go list, apart from `:91`/`:98`/`:103`, where the doc says "naming `lym config gate`". I did not check whether the `cfg.Tier` errors carry the clause themselves.
   - The old "only the prime adds" text has no remaining occurrences.

3. No finding on F3/F4. `namesProject` has three call sites, all in `dotnetscope.go` (`:158`, `:160`, `:179`), and all go through the changed function. Nothing else calls it.

Calls made:
- grep for the `fmt.Errorf`/`errors.New("gate test` sites in `tiers.go`, to list the refusals against the role-clause set. Context showed the file before the edit and not the line numbers after it.
- Same grep for `namesProject` and "only the prime adds" across the package, to check callers and leftover text.
