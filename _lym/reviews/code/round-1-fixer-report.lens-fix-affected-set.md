```yaml
lens: fix-affected-set
outcome: reported
```

Unchecked findings: none. F1 is the only finding, and its premise names a single omission. The set it implies is every link to the old anchor `#the-first-hub-after-the-pull`.

No findings.

I searched all `*.md` files, excluding `_lym/reviews`, for `the-first-hub-after-the-pull`. The only hit is `docs/research/single-repo/sandbox-3-reclone-2026-11-21/report.md:28`, which already uses the numbered form `#3-the-first-hub-after-the-pull`. No link to the old anchor remains. The new bullet in `_lym/landing/summary.md` names the same anchor.

Tool calls:
1. `grep -rn "the-first-hub-after-the-pull"` over `*.md` files: the changed files and the repo's other docs were not in my context, and F1's premise is a set of links to one anchor.
