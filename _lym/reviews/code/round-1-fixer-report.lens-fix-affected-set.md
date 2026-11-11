```yaml
lens: fix-affected-set
outcome: reported
```

No findings. The search of `docs`, `README.md` and `CLAUDE.md` for stale descriptions of `internal/machinename` and `internal/standalonestate` found only the two lines the fixer already updated (`docs/overview.md:362` and `:367`). The other hits are historical benchmark reports, which record past runs and are out of scope.

I did not check F2 and F3. They are coverage gaps in single test tables, so there is no set to search.

Tool calls:
1. `grep` over docs, README and CLAUDE.md for `machinename|standalonestate` with stale-description keywords. I used it to find other descriptions of those packages that my context lacked.
