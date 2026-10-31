```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked that every sibling decision, criterion and cross-reference the changed text touches still agrees with it.

Siblings checked: Scope, D4 Bound, D4's decision-add bullets, D5, D6's stop-marker paragraph, D13, Constraints, Auto-mode assumptions, Open risks, and the D4, D5 and D13 acceptance criteria.

- **LOW**, Scope, the `loom-discussion-spec.md` bullet ("a fence closes by CommonMark's rule"):
  - Summary: D4 now also changes how a fence opens, but the Scope line that lists the pinned spec's changes still mentions only how it closes.
  - Evidence: D4 now says "A fence opens on a line of up to three spaces of indent…" and "A line indented four spaces or more neither opens nor closes a fence". The Scope bullet still says only "a fence closes by CommonMark's rule", while `contracts/specs/loom-discussion-spec.md:52` says only "Lines inside fenced blocks are not read". That spec line changes with both rules, so its listed change should say a fence opens and closes by CommonMark's rule.
- **LOW**, D4 acceptance criterion ("one differing from its committed copy by an appended entry whose commit failed naming `lym fabric commit`"):
  - Summary: the criterion covers only the appended-entry case, while its twin in D4's body also covers appended `## PATTERN changes` bullets.
  - Evidence: the D4 bullet says "differs from the committed one only by appended `Added after Discussion` entries or `## PATTERN changes` bullets". No criterion row reaches the PATTERN-bullet case, and that case is what makes `decision add --pattern-*` on darn (D3) recoverable after a failed commit.

Tool calls:
- grep of `_lym/discussion/decision-record.md` for fence, gate, `RetiredKeys` and Darn-Write mentions: my context held only some sections of the changed record, so I needed the siblings outside them.
- grep of `contracts/specs/` for retired keys and fences, plus the Open risks lines: I had to check whether a pinned spec states the fence rule or the retired-key refusal, and read the edited Open risks line in full.
- grep of specs and recipe YAML for a pinned Darn-Write `discussion` gate: D5's guard change could leave a pinned gate list stale, and my context held no such file. The search found none.
