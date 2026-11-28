```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix's change set (summary-line spellings, line readers, wait derivation sites).

Unchecked findings: none. F1, F2, F3 and F5 have set premises and were searched; F4's required rows (slotted, carried, shared build, rerun) are all in `internal/gatecli/slotwait_test.go`; F6 is a single site.

no findings

- F5/F3: every literal spelling of the summary line outside `docs/research` is in the new form, apart from the two rows in `runlog_test.go` that spell the old form on purpose.
- F3: the patterns show as unchanged context against `main`, and I found no other parser of the line.
- F1/F2: I found no site that sets or reads the wait and was missed. `reportBlocked`, the build record and the cancelled `--copies` record still yield 0 or print no line, and `callSpan` still reads `WaitStart` unchanged.

Tool calls:
1. `git grep` for literal summary-line spellings: my context held only a narrower search from before the change.
2. `git grep` for readers and mentions of the summary line: needed repo-wide coverage, docs and stencils included.
3. `git grep` for sites that set or describe the wait: needed the post-change set of sites.
4. `git diff main HEAD` on `runlog.go`: my context did not show `main`'s patterns.
