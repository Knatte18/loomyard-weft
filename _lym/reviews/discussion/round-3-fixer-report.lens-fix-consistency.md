```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the round-3 changes to `decision-record.md` and `support-log.md` still agree with the text around them.

**Findings**

1. **LOW.** Changed text: D13's new sentence "A test that sets a process-global seam, the workspace builder included, does so through the seam's own setter … puts back what was there before it returns, and does not call `t.Parallel`".
   - **Problem:** the rule can be read as covering every test that builds a hub through `internal/hubforge`, which conflicts with a sibling.
   - **Evidence:** the sentence just above it, which this round did not change, says `internal/hubforge` installs the required seams "the first time it builds a hub, and never restores it". Read broadly, the new rule would also bind every test that triggers that first build. Under that reading such tests could never be parallel, and they would have to restore the seams that hubforge says it never restores.
   - **Fix:** say that hubforge's one-time install is outside the rule, for example "a test that sets a seam itself".

2. **LOW.** Changed text: removing `## Open risks` from `decision-record.md`, together with the Q12 and Q15 edits in `support-log.md`.
   - **Problem:** three `support-log.md` lines still refer to Open risks as a live section of the record. Line 62 says "raised to the parent under Open risks", line 73 says "kept under Open risks for the parent", and line 111 says "Open risks 1 and 3 went to the parent".
   - **Why only LOW:** these lines log earlier rounds and are true of those rounds. A reader may still look for a section that no longer exists. The Q12 and Q15 edits that pair with the removal were made.

**Checked and consistent**
- **D5 and the acceptance criteria:** D5's new core-row sentence matches the rewritten criterion.
- **Stencil ordering judge:** no mention is left anywhere in the record.
- **New package:** D9 part 5, the new Scope In bullet and D13's `internal/stencilsource` bullet name it the same way.
- **Below the "above fabric" bar:** D12's definition still names four cut modules, and `internal/stencilsource` falls above fabric under the closure rule.
- **Seam tests:** D13 no longer says stencil seam tests belong in the module that supplies them, and the adapter criterion no longer names the stencil store.
- **machinecli:** dropped from D13's list, which matches the review's note.

**Tool calls**
- One grep of `support-log.md` for stencil, ordering, installer and machinecli. My context held only lines 120–135 of that file and the parent's earlier grep for "judge", "clear" and "Open risk".
