```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward, covering each refusal, stop and error the nine cards add, and whether its named next step can succeed.

**Findings**

1. MEDIUM — Card 8 (D6/D9), the `SeamTests` reader error. Card 8 says the reader's error, "a `*GuardScanError` included, is returned", and `planContextFindings` returns it. So one misplaced or malformed `//lym:seam` line in any test file of the worktree halts Plan-Write, Plan-Burler-Fix and PR-Rework as an infrastructure fault. A resume cannot succeed, because the plan writer edits only plan files and cannot fix a test file. Card 5's `GuardMisplaced` message for a seam line names no way forward; only `SeamMalformed` has one. Evidence: card 8 Intent ("its error, a `*GuardScanError` included, is returned"; "a reader error is returned"), card 5 Intent (the `GuardMisplaced` seam error), and the existing message text in `internal/impactset#guardNamesInFile` ("is not directly above a top-level func Test line", no way forward). Proposed fix: give the seam-line `GuardMisplaced` message a way forward ("move the `//lym:seam` line into the comment block directly above the `func Test…` line"). Also state in card 8 that this halt is an infrastructure fault whose next step is fixing the named test file, not the plan.

2. LOW — Card 8 (D6), the `seam-test-unlisted` way forward has a dead end. Suppose a later card deletes or renames the seam test, or deletes, renames or moves the test's file:
   - "under Uses when … no later card targets it" is refused by `uses-later-target`, which reads every label of a later card, `Delete:` included.
   - "leave it to the later card that edits it" never covers, because only a later `Edit` or `Extends` covers.

   Only "under Edit" is left, and the finding offers that only "when the card changes it", so a writer reading the text literally has no admissible step. Evidence: card 8 Intent (the covering rule and the way-forward text), and `contracts/specs/loom-plan-spec.md:463` (`uses-later-target`). Proposed fix: let a later card's Delete, Rename or Move naming the test or its file cover the earlier card. Or extend the way forward with "else list it under Edit".

3. LOW — Card 9 (D8), the `doc-go-unlisted` way forward has the same dead end. If a later card deletes or moves `&lt;dir&gt;/doc.go`, Uses is refused by `uses-later-target`, and a later Delete does not count as coverage, since only a later `Edit` or `Extends` does. Evidence: card 9 Intent (the covering rule and the way-forward text), and `contracts/specs/loom-plan-spec.md:463`. Proposed fix: the same as finding 2.

4. LOW — Card 4 (D3), the no-commit error. "A read error, including a worktree with no commit, is a returned error." That error reaches `validate-discussion` as `loom: validate discussion: &lt;err&gt;` with no next step. Evidence: card 4 Intent (path classification), and the verb's error branch in `internal/loomcli#loomCLI.validateDiscussionCmd`. Proposed fix: either treat no HEAD as having no tracked roots (no path tested), or name the next step in the error ("commit the worktree, then re-run `lym loom validate-discussion`").

The other refusals all name a next step that can succeed from the state they leave: `decision-heading-malformed`, `decision-id-duplicate`, `decision-not-in-acceptance`, `citation-path-missing`, `citation-symbol-missing`, both `citation-symbol-ambiguous` forms, the quarry-outage error, the new `requireAbsRoot`/`requireSeam` guards, `SeamMalformed`, the card 6 scan violations and `test-tier-unstated`.

Tool calls:
- `ls` of `doc.go` under `contracts/recipes`, `contracts/stencils` and `contracts/specs`, plus a grep of the plan spec for `uses-later-target`: my context lacked whether non-Go directories hold a `doc.go` (none do, so no finding there) and the exact `uses-later-target` rule that findings 2 and 3 rest on.
