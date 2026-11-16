```yaml
lens: fix-consistency
outcome: reported
```

**Lens fix-consistency: no findings.**

I checked the change set for twins left stale and found none.

- **Stencil sentence for named runs:** `retro-template-task.md:10` says "The runs landed after {{.from}} and up to {{.until}}". It now holds for named runs too, because `From` is a second before the oldest landing.
- **Docs and flag help:** `_lym/landing/summary.md:80` (the `tools/retro` test checks every table) is now true. The `facts.jsonl` and `usage.jsonl` rows were added. `docs/overview.md:844` and the `lym retro` flag help stay correct. A future `--until` is now refused, but the flag text still says only "(default: now)", and the refusal message names the way forward.
- **`missing.jsonl` label:** `figures.jq:74` renders it as "no transcript directory (Claude Code layout)", and the test expectation matches. A repo-wide grep found no other consumer of the old "transcript directory" text.
- **Error text and test rows:** the old duplicate-slug and "name only landed runs" wording appears nowhere else. The tests match the new text: `retro_test.go` checks "within the last 60 days" and "since 2026-09-17T14:00:00Z", and the new `fileretro_integration_test.go` checks "already holds an entry".
- **`tools/retro` header:** `retro.sh` records the new `_lym/...` variables in a variable block, which `PATTERN-lym-files` allows for a `tools/` script.

One thing I did not check: whether a repo-wide enforcement test limits which packages the new `retrocli` test may import. I did not open the `testkit` or `cmd/lym` enforcement tests. The gate passed at tiers 0 and 1, so no such rule fired for that gate run.

Tool calls made (2 of 6):
1. `grep` for the old message texts and "transcript directory" across the repo, plus `until` in the docs, to find stale twins that my context lacked.
2. `grep` of `_lym/landing/summary.md` for the retro claims, plus a listing of enforcement tests that mention `hubforge`, to check the summary against the fix and the import-seam risk.
