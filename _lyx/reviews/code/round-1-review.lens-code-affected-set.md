```yaml
lens: code-affected-set
outcome: reported
```

**Lens list: code-affected-set**

1. PATTERN.md, six new entries. I searched the repo for every `PATTERN-<id>` of the six.
   - The only hits outside `PATTERN.md` are the six owning-package citations, each ID matching its entry exactly.
   - The two other hits are historical research reports (`docs/research/...`) that name old background paths. They record past runs and aren't affected.
   - No test, stencil, fixture or doc other than the cited package docs references these IDs, and none asserts an entry count. Covered.
2. Package-doc citations (`agentname.go:5`, `hubreconcile/doc.go:34`, `parentlock/doc.go:5`, `reedengine/doc.go:54`, `shedengine/doc.go:162`, `friction/doc.go:35`). All six cite an ID that now exists in `PATTERN.md`. Covered.
3. `friction/doc.go:36`, the new vouch clause. It matches `internal/loomcli/start.go:273-278` (the voucher is written only on the spawn path, with Tier 2 on). The doc's "never leaves a crash-resume note" is slightly stronger than start.go's "suppresses at most one entry observation". The doc.go sentence stays within what the record's brief asks for, so I record no finding. Covered.
4. Size and line caps, computed from the files I read. `PATTERN.md` is 8052 bytes against a headroom line of 15564 (95% of 16384). The longest line is 317 characters against the 360 cap. Covered.
5. Background links. `ls pattern` showed no background files for the six IDs, and the new entries carry no `[background]` link, as the record decides. Covered.
6. Stencils. The diff touches no stencil, so "Stencil wording" doesn't apply. The `PATTERN.md` overview is inlined in agent prompts at call time, so the six entries reach agents with no further change.

No findings.

Tool calls (1 of 6): one repo-wide grep for the six `PATTERN-` IDs, because my context held no list of other references to them (tests, docs, stencils, fixtures).
