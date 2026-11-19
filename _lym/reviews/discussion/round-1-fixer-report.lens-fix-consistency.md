```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-1 change set agrees with the sibling decisions, criteria and log entries it touches.

- **LOW.** D3, seed push step 1: "leased on the tip, or the absence, observed at the precondition check".
  The precondition list ("Every one-URL clone refuses, before creating anything") reads only the board branch, `lym-bolt` and the default branch's `.lym-warp`.
  No check there reads the prime's records branch, `refs/lym/heads/<prime-branch>-weft`, so nothing records the tip the lease names.
  Its name also depends on the code store's checked-out branch, and D1 says that branch is known only after the code store is cloned.
  Evidence: D1 says "named from the code store's checked-out branch", while D3's precondition paragraph observes no records-branch tip.
- **LOW.** In `support-log.md`, three earlier lines were left as they were and now disagree with the changed D3 and D4 without saying they are superseded:
  - Turn 4, `walkthrough 2`: "fixed: the seed push runs before `CloneAndWire`";
  - Turn 4, `interactions 3, bound-breaker 4`: "D3 refuses an origin holding `lym-bolt` or a root `.lym-anchor`";
  - Turn 2's git facts: "`git push origin --delete <branch>` fails under the mapping, the full ref name works", which is the evidence F3 replaced in D4.

  The new `### Round 1` entry gives the new outcomes for F1 and F2, but nothing tells a reader that these lines no longer hold.
  The entry also never covers the F3 evidence swap: its F3 line names only the `ls-remote` check.

I made no further tool calls.
