```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F3 change set for agreement with its surroundings (fix-consistency lens).

**Findings**

1. **MEDIUM** — `contracts/stencils/webster/webster-body-implementer.md:58-59` (new sweep sentences) contradicts line 43–44.
   - Line 43 says "Two deviations are sanctioned" (the guard-step fix and the failure fix). Line 44 adds "No other change outside the targets is made."
   - Line 58 now sanctions an out-of-target edit to hits that reference a retired name, and line 59 says it is not a failure fix. That makes it a third sanctioned deviation, which line 44 forbids.
   - The F1 fix offered two routes: keep the numbering, or extend line 43 to name the sweep edit as its own bounded case. The fixer reworded line 58 but left line 43 at "two", so the text now disagrees with itself.
   - Line 71 ("the first sanctioned deviation in step 2's rule") still holds.
   - Fix: extend line 43 to three sanctioned deviations, naming the retired-name sweep edit as its own bounded case, or have line 58 explicitly say it is covered by line 43.

2. **LOW** — `webster-body-implementer.md:56` (the added "same grep" sentence) refers to "the rename paragraph above", but the sweep text is now a loose sub-paragraph. The reference works only because that paragraph sits at line 49 in the same numbered item, so it is acceptable. The F3 point about `git mv` file renames still gets no word on whether the sweep applies. The review only raised this as a note, so I'm recording it here without asking for a change.

3. No other twin of this text exists. The grep over the tree (excluding `_lym`, `.git`, `.lym`) found no other stencil or recipe restating the sweep or the sanctioned-deviation wording. Hits in `tools/sandbox/SANDBOX-*-SUITE.md` concern an unrelated black-box rule.

4. The test change in `template_test.go` is consistent. Its case title is reverted and the remaining `inOrder` entries still match the stencil text.

**Tool calls**
1. `Grep` — failed because the tool is unavailable here.
2. `grep -rnE` over the tree for the sweep and sanctioned-deviation wording — to find twins and sibling references that my context lacked.
