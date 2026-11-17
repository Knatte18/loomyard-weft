```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content in the decision record that should move or go.

- MEDIUM — Notes for the plan writer, second bullet. A required edit sits in the optional Notes section: the burler review file and the fixer-report must join `PATTERN-sole-parsers`' background list in a Webster card. The plan writer may skip Notes, so the completeness-before-leanness test says this belongs in `## Constraints`. Fold it into the existing `PATTERN-sole-parsers` constraint line. Evidence: the rubric's "Extract that into `decision-record.md`'s own Decisions or Constraints first", and the spec calls Notes "never a completeness requirement".
- MEDIUM — Open risks, first bullet. It repeats D4's own `Bound:` line almost word for word ("a defect round 1 missed in untouched code stays unraised"), and the entry asked for that bound, so it carries no open risk. Cut it. D4's bound already states it. Evidence: say-it-once, and Open risks is for questions the parent must weigh.
- LOW — Constraints, `PATTERN-config-layers` and `PATTERN-lym-files` lines. Both repeat what D6 and D5 already say: that the key is a new loom template key, and the cache-degrades-to-full-round behaviour. Trim each to just the PATTERN compliance fact, or cite the decision. Evidence: D6 states "classified `CategoryNone`"; D5 states "A missing cache makes the next round full (D4)".
- LOW — `## Auto-mode assumptions`. Every bullet restates a choice its Decision already makes in full. The section is required, but its bullets could name each picked option in a few words and leave the full statement to the Decision. Evidence: compare the D4 file-level/hunk-level bullet with the D4 text "the check holds the file bound, and the hunk bound is the reviewer's".
- LOW — D2, first paragraph. "in place of today's clean-room text that tells the reviewer to form its own findings first and read prior files only after" narrates current behaviour. "replaces today's prior-rounds text" is enough. The history is already in support-log Turn 1.
- LOW — Notes for the plan writer, third bullet. "This task changes no judge text beyond what the new review keys need" is vague and gives the plan writer nothing to act on. Either name the judge-stencil change, if one is needed, as a Decision, or cut the bullet.
- LOW — Notes for the plan writer, first bullet. The fact that review start equals fix start, because the reviewer never writes the target, is a premise D5 and D6 rely on to compare snapshots. Consider stating it in D5's rationale rather than in optional Notes.

No further tool calls made.
