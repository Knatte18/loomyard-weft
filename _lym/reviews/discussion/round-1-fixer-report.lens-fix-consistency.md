```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: I checked whether the fixer's change set agrees with the sibling decisions, criteria, cross-references and twins it touches.

1. **LOW**: `## Auto-mode assumptions`, D4, D10 and D12 lines.
   The fix made new auto-picks and left the list unchanged: the fenced `markdown` block for the board body, the per-row handling of in-flight runs with Darn-Write's pre-session refusal, the map from Tapestry-Preflight to Preflight, and the plan-perch check in the `tapestry` load.
   Evidence: line 218 still reads "D4: the row name `Record-Write`, its place after Preflight, and the support log it writes."
   Line 222 names only "the `batcher.yaml` load in Preflight", and there is no D12 line for where the plan perch's budget is checked.
   D6's line was updated, to "the stop marker as the writer's halt", so the list is updated for one changed pick and not the others.

2. **LOW**: D10's new Bound sentence, "A run's current row is Preflight on a fresh seed, on a resume of a run blocked at Preflight, or after `lym loom goto --to Preflight`".
   Line 150 of the same fix adds a fourth route, "`tapestry`'s retired-row map sends a run parked on Tapestry-Preflight to Preflight", and the list leaves it out.
   That remapped run has a Tapestry-Preflight entry in its history, so it is exactly the kind of run the Bound claims to enumerate.

3. **LOW**: Goal and D1 now say "build segment", following the `Added after Discussion` entry, but a twin of the old wording stays.
   D5, line 97: "The darn writer row keeps its `discussion` gate".
   The entry, line 198, says "The contract and the code do not call this slot 'the writer'".
   D1's clause and its acceptance criterion were renamed; D5's reference to the same Darn-Write slot as "writer row" was not.

4. **LOW**: Constraints, the PATTERN-refusal-way-forward line, line 206.
   F6 gave D6's stop halt a way forward ("The halt's way forward is the orch or the operator recording the answer with `lym loom decision add`, then `lym loom resume`").
   The line still lists only "D1's contract refusal, D2's non-loom refusal, D3's verb refusals, D4's missing-record halt and D13's retired-key refusal", so it does not cover the D6 halt, and the line's promise that "a test reaches each" leaves it out.

Tool calls:
- `cat -n` of decision-record.md lines 1–140: my context held only a truncated preview of the changed record, because the full output was saved to a file.
- `cat -n` of decision-record.md lines 141–260: same reason, for the rest of the record.
