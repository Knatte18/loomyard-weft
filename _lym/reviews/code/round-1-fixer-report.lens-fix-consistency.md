```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fix change set against its surroundings.

**BLOCKING: `_lym/landing/summary.md`, `## Merge danger`, the rewritten `**Door:**` and `**Blast radius:**` lines.** Both lines now fail the description validator and contradict the spec, where the old lines passed.
- `internal/summaryparser/description.go`, `mergeDangerFindings`: the door value must equal exactly `one-way` or `two-way`, and the blast radius must satisfy `len(strings.Fields(radius)) == 1`.
- `contracts/specs/loom-final-summary-spec.md`: "**Blast radius** is one word naming what breaks if the change is wrong", and "`lym loom validate-description` and every gate that checks a change description refuse one without the section, a door other than those two words, and a blast radius other than one word."
- The new door line reads "one-way once a layer holds `unattended`; two-way while no layer sets it, …", and the radius is a full sentence.
- F2's `fix` asked for this prose in the labelled lines, so the proposal was wrong in that respect. The two lines need to be bare values (for example `**Door:** one-way` and `**Blast radius:** config`), with the explanation as separate lines below them in the section.
- `lym loom validate-description` was not run by me.

**LOW: `internal/beamcli/dobbydeps.go`, `tellRefusal`.** F1's twin in the same function was left as it was: `logger.Warn("beamcli: a refused autonomy setting was not posted to the orch", …)` still names the key informally.
- The fix changed the mail to name `beam.autonomy` through `autonomyName()`.
- `PATTERN.md:17` (`PATTERN-config-key-names`) covers "Naming a config key in a message, comment, doc or stencil: in full".

**LOW: `internal/beamcli/dobbydeps.go`, `stamp` doc "telling the orch of a refused value first", and the summary's "the first stamped mail is preceded by one mail".** With the post now outside the lock, this no longer holds for concurrent stamps of one run.
- The first caller records `told[slug]` and unlocks before `s.post`.
- A second caller sees the entry, returns at once, and can send its stamped mail before the refusal mail lands.
- If the first post then fails, that second mail went out with no refusal told at all.

Tool calls:
1. grep for the Merge danger labels: my context lacked the validator and spec for the summary's format.
2. Read `description.go` and the spec section: my context lacked the exact rules.
3. grep for `PATTERN-config-key-names`: my context held only the review's paraphrase of it.
