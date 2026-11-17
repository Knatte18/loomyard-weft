```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 fix set agrees with the sibling text it touches, and find any twin of a changed passage that was left as it was.

- **LOW** — Acceptance criteria, the `Review:` criterion ("`pattern/PATTERN-gate-slot.md` names the map run as holding the larger of tier 0's and tier 1's slot asks").
  F7's clause about an every-slot tier was added to D7 and to the Constraints bullet, but the criterion that checks the background's wording still uses the old phrase.
  Evidence: D7 and the `PATTERN-gate-slot` constraint now end "where a tier asking for every slot ranks above any count"; the `Review:` criterion does not.
- **LOW** — D8 body, "Two map runs over the same package from the same worktree each write a whole map, and the later write wins."
  F3 changed D2's first sentence and D8's Rationale, but D8's own body sentence about the race still says each run writes a whole map.
  A failed build's write keeps entries it read, which may come from the other run.
  Evidence: D8 Rationale now reads "a failed build's write keeps the entries it read, which may come from the other run", one paragraph below the unchanged body sentence.
- **LOW** — D4 Rationale, "clamping the per-process caps to the tier's `memory_mb` gives the same total as a scope around them."
  F4 limited D4's body bound to hosts whose memory cap covers a process tree, but the Rationale still states the equivalence without that condition.
  Evidence: D4 body now says "on a host whose memory cap covers a process tree; on one where it does not, one test runs at a time and each process is capped alone".
- **LOW** — D6, the new sentence "A build whose listing of the linked packages fails is checked before its failure is written, for HEAD and for the files the package itself owns".
  The fix covers a failed listing, but D6's opening still places the first check after the build "has listed the linked packages and compiled".
  So a build that lists and then fails to compile still has no stated check before D7 writes its failure.
  Evidence: D6's first sentence, "Once a package's coverage build has listed the linked packages and compiled, and again before its map write", next to D7's "A failed build records the failure on the package's map".

I made no further tool calls; everything came from my context.
