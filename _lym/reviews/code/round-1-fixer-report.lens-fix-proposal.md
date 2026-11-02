```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F2, F3, F4) and report breakage or counter-cases. F1 was fixed with the fixer's own change, so it is outside this lens.

**No findings.**

- **F3, `target` built before `reconcileAtWake`.**
  - The reconcile now recognises the daemon's own hub entry when `r.Owned` is nil and `kept` is set. The row clears `Owned` in `onSleep`, so with the old order `reconcileCalls` would be 0 and the assertion would fail.
  - When both `r.Owned` and `kept` are nil, the behaviour is unchanged, as the "hub entry live before the request" row shows.
  - An operator's hub entry cannot be mistaken for the daemon's. `reconcileAtWake` also compares `PausedAt`, which differs, so the skip-and-name-the-resume path is unchanged.
  - `fail(deps, r, kept, …)` still receives the unmodified `r`.
- **F2, newline routing row and unwritable-file row.**
  - The newline row's stop text is short but multi-line, so only the `strings.Contains(line, "\n")` branch can route it to the file. A missing newline condition fails the row.
  - The unwritable-file row relies on `MkdirAll` under a regular file failing with ENOTDIR, which is deterministic. It uses an over-bound stop, so the write is attempted.
  - Both rows leave the notice with no newline and within the bound, so `requireNotice` passes.
- **F4, post-lift write failure row and long-reason fail row.**
  - The broken request file makes `finish` fail while `target` is held in memory, so the lift and the notice still run. The row passes only if the error is returned and the entry is lifted.
  - `"job refused "` repeated `noticeLineBound/8` times is 1536 bytes, over the bound, so the file route is taken. The headline asserted, `The hibernate request failed; its pause was lifted.`, matches the `fail` headline text.
- **Unchecked.** I did not check whether `liftOwned` reads the request file between the reconcile and the lift, which would break the F4 row. The gate passed tier 0 for the engine, so the row runs green.

Further tool calls: none. My context held every changed file and the gate result.
