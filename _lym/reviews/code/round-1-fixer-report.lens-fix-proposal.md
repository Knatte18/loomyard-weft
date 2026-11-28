```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F7).

**LOW**
- **Changed text:** the F3 exception, added only to the `ownPackageTiers` doc comment in `internal/gatecli/scope.go`.
- **Summary:** F3's proposal asked for that one comment alone, so the other statements of the narrowing rule still say every higher tier that does not cover tier0 runs the cards' packages alone.
- **Evidence:** a `carry: false` tier, any tier of a `--no-carry` call and a command with no target placeholder are now left out of the narrowing, and they run whole. The unqualified rule stands in:
  - the `computeScope` doc comment ("Under the batch key, each listed tier above tier0 that does not cover tier0 runs the cards' packages alone")
  - `internal/gatecli/doc.go` ("Each other listed tier runs the cards' packages alone … and the call names it on stderr")
  - the per-batch bullet of `contracts/specs/loom-plan-spec.md`, which F5's new "scoped as its bullet below states" now points readers to
  - F6's new `--card` help text
- **How much it matters:** `doc.go` says elsewhere that a `carry: false` tier runs whole, so a reader can reconcile most of this. The sharpest case is the "names it on stderr" claim, because a whole-running tier is no longer named by the batch notice.

I found no breaking case for the other proposals:
- **F1:** the same string value reaches every replaced site, and `stopKindBatch` is untouched.
- **F2 and F4:** dropping `own = withMainPackages(...)` would change tier1's invocation, and so would using `sc.skip` in place of `ownSkip`. The row would fail either way.
- **F3 behaviour:** a tier left out of the narrowing takes the dependents' `listedPackages` and watch skip, as it did before the branch's diff. Its test matches what `planTierRuns` and `scopeInvocations` treat as running whole.
- **F5, F6 and F7:** nothing else breaks.

Tool calls: none; my context already held everything this lens needed.
