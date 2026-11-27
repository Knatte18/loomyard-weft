```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check of the F2–F4 changes.

Unchecked list: F1 (deferred, so no fix to check; its reader set was not rerun). F2, F3 and F4 were checked.

- **LOW** — F3's twin in the change description, `_lym/landing/summary.md:15-16`.
  - The text reads: "The end line carries both the wall-clock time waited and the awake time, so a nine-hour gap like the one in the report shows whether the lock was held that long or the host was suspended."
  - It keeps the unqualified claim that F3 weakened in the `ensure` doc comment. A gap can also come from a wall-clock step, and the suspend reading holds only where the monotonic clock pauses in suspend.
  - The fixer may not be allowed to edit this file under its `_lym` rules. If so, the report should name it.

F2: the search found no other repository text under `internal`, `contracts` or `docs` that calls a `transient-stop` halt a network or host failure. The three named sites are all reworded.

F4: the build skew over transient pair is now pinned, and rows already cover interrupted against each of the two. No member of the set is missing.

Tool calls:
1. One `grep` over `internal`, `contracts`, `docs`, `tools`, `cmd` and `_lym/landing` for the F2 and F3 claim wording. My context had only the branch diff and the earlier `transient-stop` search, not a search for twins of these claims.
