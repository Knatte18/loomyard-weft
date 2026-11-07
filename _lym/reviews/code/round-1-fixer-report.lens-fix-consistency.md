```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The four edits agree with their surroundings:
- **F3, comment and doc:** the `Registration.Build` comment (`template.go:75`) and `doc.go:57` now carry the clause that `PATTERN-test-isolation.md:202-203` already states.
- **F4, message:** no other copy of the "take CopyHub" advice exists. The only match is the changed line at `template.go:357`.
- **F1 and F2, test rows:** they match the comments that say a build fails on a skip and records a failure. `wantFirst` of `Skip` is also a substring of `Skipf`'s refusal message, so the first-caller assertion does not tell the two rows apart. That is an assertion-strength point, not an inconsistency, and it is outside this lens.

Tool calls:
1. Ran `grep` across the worktree for "take CopyHub", "once-only registration" and "Bolt replay". My context did not hold the `PATTERN-test-isolation.md` wording or any other copy of the message.
