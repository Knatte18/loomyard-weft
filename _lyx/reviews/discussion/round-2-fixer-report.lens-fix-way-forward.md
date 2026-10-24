```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the fix's new ways forward (fallbacks, refusals, next steps, tests) through the fix-way-forward lens.

- [MEDIUM] The fix leaves no outcome for a card whose Targets are all inert. Changed text: the Decision 3 `--card` bullet, "a Target whose file is inert (Decision 5) is dropped from the seed", and the matching card acceptance criterion. If every Target of a card is inert, such as a docs-only card, the seed is empty. The record's empty-scope cases don't cover that state:
  - Decision 2's result bullet distinguishes only "a non-empty seed that selected no test class" from "an empty or inert-only diff".
  - Decision 3 handles the empty diff, the inert-only diff and the empty test scope "from a non-empty seed".
  - Decision 6 skips the build only for "an inert-only or empty-diff empty scope".

  An all-inert card is none of these, so whether it runs nothing, runs the build or runs whole is left undecided. Evidence: Decision 2's result bullet ("apart from an empty or inert-only diff"), Decision 3's "A closure that selects no test class from a non-empty seed", and Decision 6's "inert-only or empty-diff empty scope runs no build".

I found no other issues. The other new ways forward lead somewhere and can be reached from the state they name:
- **Selected class without a single `.csproj`:** falls back to whole, with the reason on stderr.
- **Empty test scope:** an opted-in tier runs the build, and a tier not opted in runs whole.
- **Untracked files:** read through `ls-files`, which impactset already uses.
- **tier1 fixture:** two parallel projects built with `-m`.
- **Landing-tier `filter`:** `docs/dotnet-hub.md` tells the hub what to do.

No further tool calls made.
