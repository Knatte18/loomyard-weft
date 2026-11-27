```yaml
lens: fix-affected-set
outcome: reported
```

Task: checking that the fix covered every member of each set-premised finding (F1, F2, F3, F7, F8, F10).

**Unchecked findings:** none of the set-premised ones. F4, F5, F6, F9 and F11 have single-site premises, so I had no set to rerun for them.

**Findings:**

- **LOW. F10's twins** (the `finalize.go` `Call` doc, the paragraph "The lock is freed before the fixer or a fix review starts"; `docs/overview.md`, the `Finalize` bullet, "It then retakes the lock at the front of the queue. It merges the parent anew and verifies under the lock").
  - Both still describe going from the fixer straight back to the locked steps.
  - The reworded `freeLock` doc now says "The run then runs its passes again, the lock-free pass first while lost races remain".
  - `Call`'s loop runs `gateUnlocked` before `landUnderLock` whenever `lostRaces < FinalizeLostRaces`.
  - The `Call` doc text is "Once their change is let through, Finalize takes the lock again at the front of the queue, and it runs every step under the lock again, from the pull-request step on."
- **LOW. F3's new recipe comment line** (`darn-recipe.yaml:308`, `sampler-recipe.yaml:264`, `tapestry-recipe.yaml:531`, `hardener-recipe.yaml:179`).
  - "After landing.finalize_lost_races lost races, and over a parent already holding the work, it merges and gates under the lock." is about 125 characters of text.
  - The repository's line rule is under 120, so it should break at the comma before "it merges".

**Covered with nothing further:**
- **F2:** both `parentMergeTip` callers pass `front`.
- **F3:** the search of contracts, docs, patterns and Go sources found no other "takes the lock first" wording. The exception is `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml`, a deliberately frozen copy.
- **F7:** the other "before any parent-side mutation" claim (`deps.go:344`, `EnginesRun`) still holds.
- **F1:** `PATTERN-landing` and `PATTERN-owner-machine` are not made stale.

**Tool calls:**
1. `git grep` for lock-order wording in contracts, docs and patterns, because my context held only the earlier search from before the edits.
2. `git grep` for the same wording in Go sources, for the same reason.
3. `git grep` for config-notice timing claims, because my context had no repo-wide search for them.
4. Read the head of `pattern/PATTERN-landing.md` and `deps.go:340-347`, neither of which was in my context.
