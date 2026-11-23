```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the `## Merge danger` edit in `_lym/landing/summary.md` agrees with its surroundings and with the final-summary spec.

Findings:

1. **BLOCKING** — `_lym/landing/summary.md`, `## Merge danger`, the `**Door:**` and `**Blast radius:**` lines.
   - **Problem:** the edit breaks the section's format, so `lym loom validate-description` and the gates that check a change description will refuse it.
   - **Door:** the line is now `**Door:** one-way.` The spec says the value must be exactly `one-way` or `two-way`, optionally in backticks (`loom-final-summary-spec.md:40`, `:47`, `:49-50`). The trailing period is not a valid value, and the explanatory lines that follow it may break parsing too.
   - **Blast radius:** the value is now a multi-word phrase, "dobby and the orch's beam pull request upkeep", plus a path. The spec says it is "one word" and refuses any other (`loom-final-summary-spec.md:44`, `:50`).
   - **Proposed fix:** write `**Door:** one-way` and `**Blast radius:** dobby`, or another single word. The rationale about the suppressed notice and the dropped `member` key belongs in `## Design decisions` or in prose above the section, not in the two lines.

2. **MEDIUM** — same section.
   - **Problem:** the review's `fix` asked for a widened blast radius, but the spec's one-word rule means it cannot be written as asked.
   - **Proposed fix:** pick one word that covers dobby and the orch's upkeep, such as `landing`, or keep `dobby`. Record the widening in `## Design decisions`. The existing last bullet there already states the orch dependency on `land_notice`, so it can carry the "one-way" rationale.

The other siblings agree with the change:
- `## What changes` says the old `member` key is ignored on load. That matches the claim that `LandNoticeState` no longer writes it.
- `contracts/orchrole/role-beams.md:55-56` matches the new blast-radius text.

Tool calls:
1. `grep` on `_bolt/_lym/specs/loom/loom-final-summary-spec.md` for the Merge danger format. My context lacked that spec.
