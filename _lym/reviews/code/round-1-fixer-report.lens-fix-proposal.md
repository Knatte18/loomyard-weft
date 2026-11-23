```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposal behind F1, the `## Merge danger` rewrite in `_lym/landing/summary.md`.

**Findings**

- **BLOCKING** — `_lym/landing/summary.md`, `## Merge danger`.
  - **Summary:** the rewritten section fails `lym loom validate-description`.
  - **Evidence:** `loom-final-summary-spec.md:49-50` says validation refuses a door other than `one-way` or `two-way` and a blast radius other than one word.
    The fixer's `**Blast radius:** dobby and the orch's beam pull request upkeep: an orch no longer gets ...` is many words.
    The `**Door:** one-way.` line carries a trailing period, which may also fail the exact-word check.
    The extra prose lines between the two keys may break parsing too.
    Spec line 44 gives the blast radius as "one word naming what breaks", so the reviewer's proposal to widen it to "dobby and the orch's beam pull request upkeep" cannot be applied literally.
  - **Fix:** write `**Door:** one-way` and `**Blast radius:** dobby` (or one other word), and put the explanation in prose above the section.
    The review's wish to widen the radius cannot be met inside the one-word limit; if `orch` is the better word, use `orch` or `dobby`, not a phrase.

- **LOW** — the door call itself.
  - **Summary:** the spec defines one-way as "a revert of the landing commit does not undo the change's effect".
  - **Evidence:** the review's own Issue text says "A revert stays safe", so the spec's definition does not make this one-way by revert-safety alone.
    The review's grounds are the rubric's rule that a setting-meaning or record-format change is one-way.
    Spec lines 41-43 list a record format another binary reads, so the dobby state record losing `member` supports one-way, but the old binary re-reads membership from the board.
  - **Fix:** keep one-way, since the rubric mandates it, but this is a judgment call the fixer should state in the report.

Calls made: 1.
- Bash `grep` of `loom-final-summary-spec.md` for "door", to learn the validated format of the Merge danger section, which my context lacked.
