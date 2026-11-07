```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 changes agree with the sibling decisions, criteria and cross-references they touch, and look for a changed passage whose twin was left as it was.

no findings

Checked:
- **D2's new upstream-tip definition** agrees with what depends on it: D7's list of steps that stay in the test, D7's `endedHub` bullet, the two D2 criteria, D3's Bound and D3's first criterion. After D3's push, the tracking ref and the bare's tip are equal, so "HEAD equal to its upstream" in D3 still holds under D2's definition.
- **D2's Bound now also names seam setters and once-only registrations.** D7's "a process-wide call D2's Bound names" picks them up, and both new D7 review criteria match the Bound.
- **D1's walk-up and missing-directory rule** has its matching criterion. It does not conflict with the reftable porcelain fallback.
- **The `3aca2e50` wording** reads the same in Scope Out, D10 and Open risks. The support log's Turn 4 and Q-ledger lines are history from earlier rounds, so they are not stale twins.
- **The support log's new "Round 2" entry** matches the five dispositions in the record.

Further tool calls: none.
