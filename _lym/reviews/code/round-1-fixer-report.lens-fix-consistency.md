```yaml
lens: fix-consistency
outcome: reported
```

no findings

I checked the four changes against their twins:
- The C# spec pointer in `webster-template-plan-fix.md` has no twin in the other stencils that cite the checks spec.
- The `ImpactSummary` wording in the rubric now matches `loom-plan-spec.md:84` and the table at line 116. The rubric's own heading, "A `Rename`, `Move`, `Prosa`, or `Custom` card", still fits.
- The new entries in `loom-plan-checks-spec.md` don't contradict the stencils that cite it (`loom-block-plan-format.md`, `loom-block-plan-cards.md`).

Tool calls:
1. `grep` for the `Edit`/`Delete`-only wording and for citations of the checks spec across the stencils and `internal`, plus `ImpactSummary` in `loom-plan-spec.md`: to find twins and cross-references that my context lacked.
