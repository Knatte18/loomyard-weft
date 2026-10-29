```yaml
lens: leanness
outcome: reported
```

Lens: leanness. Find what to move out of or cut from the decision record.

**Findings**

1. **MEDIUM — D1 Rationale, fourth bullet ("A block always adds a `hook_blocking_error` attachment…").**
   This bullet argues against an option the record did not pick, which is rejected-alternative material.
   The support log already holds the same point under `## Rejected alternatives` (the first two bullets).
   Completeness test: what the plan writer needs is "no `decision` or `reason` key", and D1's decision text and its acceptance criterion already say that.
   Cut the bullet from the record, or shrink it to one clause saying a block always adds the command text and the error label.

2. **LOW — D2 Rationale, first sentence.**
   The sentence on the PostToolUse block leaking the command text is the reason for rejecting the block, and the support log carries it again ("Keep the block on PostToolUse…").
   The remaining sentences in D2's rationale are about how the chosen output renders, so they stay.
   Trim the first sentence to a clause.

3. **LOW — Constraints, the bullet on `c49ba448`.**
   The history of why that commit chose the block is deliberation narrative.
   The part the plan writer needs is that existing docs and tests state the block design and change with it, plus the tier3 proof, and acceptance criterion D1 (second bullet) already covers the proof.
   Cut it to "`c49ba448`'s block design, as stated in the docs and in `TestHookContext`, is replaced", and move the rest to the support log.

4. **LOW — Constraints, the `PATTERN-daemon-wakeups` and `PATTERN-test-speed` bullets.**
   Both restate standing rules without adding anything specific to this task.
   The daemon-wakeups bullet says only that nothing is added, so it fails the cut test.
   The test-speed bullet repeats the PATTERN entry word for word.
   Cut both.

5. **LOW — Scope "Out", the "Engines other than Claude" bullet, against the `PATTERN-shuttle-provider-seam` Constraint.**
   The same rule is stated twice, so keep one.
   The Constraint is the better place because it names the entry.

6. **LOW — Auto-mode assumptions, against Scope.**
   Two of the three assumptions repeat what Scope already says: the tool-boundary path is in Scope "In", and leaving the bell untouched is in Scope "Out".
   Only the reading of "is logged" is a real assumption.
   Keep that one and drop the other two, or move their one-line reasons into Scope.

7. **LOW — Open risks, fourth bullet ("A ring that races a turn the operator starts…").**
   The bullet itself says this is what the Stop path already does, and no decision or action depends on it.
   Cut it, or move it into D3's Bound if it is meant as a bound.

8. **LOW — D3, the Bound sentence on what Claude Code 2.1.312 does with stderr ("reads an asyncRewake hook's stderr after the process exits…").**
   It is evidence for the bound and can stay.
   It partly overlaps D4's rationale on the same rewake rendering; one shared clause would do.

**Other things noticed, outside this lens's focus**

- **LOW — D4.**
  The chosen `rewakeMessage` ends in a colon, and the mail lines that follow it start with `[creel]`.
  The model would read `[creel] Mail that arrived while you were idle: [creel] #12 from …`, with `[creel]` twice.
  Dropping `[creel]` from the prefix would avoid that.
- **LOW — Acceptance criteria for D1 and D2.**
  "The transcript holds no hook command text" depends on the test knowing the command string.
  That is fine, but it overlaps the JSON-shape criterion.
  This is not a cut.

I made no further tool calls; everything needed was already in my context.
