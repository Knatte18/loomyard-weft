```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran each set-shaped finding's search to check the fix covers every member.

Unchecked: none.

No findings.

- **F1 (the landingshed import allowlist):** the allowlist in `internal/landingshed/seam_enforcement_test.go` already has `shuttleengine`, `segmentcolor`, `stencilstore`, `modelspec` and `shedtransient`, the other packages `ensureSession` needs. `pullrequest` comes from card 11. `runproc` is only used by loomcli's `RegisterSession` wiring, so landingshed does not import it. `languageconfig` is ruled out by the plain append the fix added. With `stencil`, `parentdirective` and `editdirective` added, the set is complete.
- **F2 (entry `Status` read followed by a wait poll):** card 11's step 6 is the only entry path that reads `Status` and then hands off to `wait`. Step 4 (the no path) returns Done without waiting. The fix covers step 6.
- **F3 (comments claiming PR-Review never returns Stuck or always returns Done):**
  - The two header sentences, `darn-recipe.yaml:27` and `loom-recipe.yaml:37`, are now named in card 11.
  - The row comments `darn-recipe.yaml:221` and `loom-recipe.yaml:535` ("stub seat … always returns Done") fall under card 11's existing "every header comment calling the row a stub that always returns Done" sentence.
  - `internal/loomrecipe/doc.go:16` is rewritten in card 11 line 104.
  - `docs/overview.md:578` is rewritten in card 11 line 103.
  - The pawl recipe has no such claim.

Tool calls:
1. Read the `landingshedAllowedImports` list. My context had the allowlist file only through grep hits for stencil, parentdirective, editdirective and languageconfig, not the full list.
2. Grepped the recipes, docs and package docs for never-Stuck and always-Done claims, to rerun F3's search across the whole set.
3. Grepped plan cards 11 and 00 for "stub". I needed to know which plan text covers the row comments, and my context had card 11 only from line 40 on, plus line 94.
4. Grepped the repository for PR-Review comments about Done, Stuck or `on_stuck`, to widen F3's search to other wordings.
5. Grepped the plan for `loomrecipe/doc.go` and "always ends Done". This checked that the `loomrecipe` doc sentence turned up in call 4 is covered.
6. Read card 11 line 104, to confirm that rewrite in full.
