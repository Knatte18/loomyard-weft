---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/plan/07-test-tier-check.md:43
    summary: "The test-tier-unstated way forward never offers the support-only form: a TestMain or helper file is told to add a go:build line, and untagged appears only in the drop clause"
    fix: "In card 7's Intent, at the sentence quoting the way forward (line 43), change the quoted string to open with: add an Intent line naming the file and its `//go:build` line, such as `//go:build tier0`, or the word `untagged` when the file holds support only (TestMain, helpers) and carries no build line, on a line that names no other test file; keep the existing clauses that follow. Keep the same string in the loom-plan-spec.md row card 7 adds. In the D7 rows (line 51), assert that the Detail of the no-statement row holds the untagged clause, and keep the existing untagged passes row as the fix shown passing."
  - id: F2
    severity: LOW
    class: scope
    location: _lym/plan/07-test-tier-check.md:51
    summary: The D7 rows pin the one-line failure and the split-line fix for two tagged files, but not for an untagged helper beside a tagged test, where the carry rule makes the helper's own untagged statement fail
    fix: "In card 7's D7 rows, extend the row for a card creating `internal/x/a_test.go` and `internal/x/b_test.go` with a case for a card creating `internal/x/x_test.go` (tier0) and `internal/x/helper_test.go` (untagged): a single Intent line naming both files fails both with `test-tier-unstated`, the Detail naming tier0 and the own-line clause, and the same two statements split over two Intent lines pass. Add it as a row of the existing table, not a new test function."
  - id: F3
    severity: LOW
    class: decision
    location: _lym/plan/07-test-tier-check.md:42
    summary: The pass rule says the lines name exactly one tag but does not say whether tags are collected as a set or counted per mention, so a file named on two lines that both state tier1 may fail or pass
    fix: "In card 7's Intent, state in the sentence that collects the tags that they are collected as a set of distinct tags, so the same tag on several naming lines is one tag and passes; carry the same sentence into the loom-plan-spec.md row. Add a row to the D7 table in which one file is named on two Intent lines that both state `//go:build tier1` and passes."
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/07-test-tier-check.md:58
    summary: Cards 7, 8 and 9 each edit validatePlanCmd's Long help without saying which sentence they change, and the existing sentence ends with a clause that is false for the three new checks
    fix: "In card 7's Docs paragraph, name the sentence it rewrites, the one starting Every mode but --require-approved also runs rename-tool-missing and pattern-changes-mismatch, which read language.yaml and the decision record, and say the rewrite lists test-tier-unstated and moves the which-read clause to the two checks it describes. In cards 8 and 9, say each adds its check name (seam-test-unlisted, doc-go-unlisted) to that same sentence. The refs stay typed Edit on all three cards, since each rewrites existing content."
---

### [MEDIUM] The test-tier-unstated way forward never offers the support-only form

**Location:** `_lym/plan/07-test-tier-check.md:43`

**Issue:** The decision record's D7 admits a created test file that states `untagged`, "for a support-only file".
`PATTERN-test-speed` says an untagged test file holds support only, such as `TestMain`, and the repo's `internal/loomshed/testmain_test.go` is such a file.
The way-forward string card 7 prescribes tells a writer to "add an Intent line naming the file and its `//go:build` line, such as `//go:build tier0`".
For a card that creates a `TestMain` or helper file, that instruction cannot be followed: a support-only file carries no build line, and tagging a `TestMain` file `tier0` would drop it from the `tier1` build.
The word `untagged` appears in the string only in the clause "drop untagged or the tag when both appear", which reads as a rule for a conflict, not as the form a support-only file takes.
A writer who obeys the string writes "no build line" for the helper, which names neither a tag nor `untagged`, and gets the same finding again.
The rows reach the `untagged` pass only by the row text, and a failing writer does not read the rows.
The spec row would hold the form, but the finding text is what the failing writer reads first.

### [LOW] No row for an untagged helper beside a tagged test

**Location:** `_lym/plan/07-test-tier-check.md:51`

**Issue:** The carry rule means a single Intent line naming `helper_test.go` (untagged) and `x_test.go` (tier0) gives each file both `untagged` and `tier0`, so the helper's own correct statement fails.
This is the realistic way a writer words a card that creates a test and its helper, and the fix, one statement per line, is pinned only for two tagged files.
Adding it as a row of the existing table keeps the test economy: it is a state the card names as failing (untagged plus a tag, reached through the carry) and today no row shows its fix passing.

### [LOW] Set or count of tags is left open

**Location:** `_lym/plan/07-test-tier-check.md:42`

**Issue:** "The file passes when the lines name exactly one tag" does not say whether two naming lines that both state `tier1` name one tag or two.
D7's "Together they name exactly one tier tag" reads as a set, but the card's own `||` rule speaks of "each pair of consecutive ones", which an implementer can read over mentions.
A count would fail a file stated twice with the same tier, and the way forward gives that writer nothing to fix: join with `||` is wrong and there is nothing to drop.

### [LOW] Long help sentence for the three new checks has no owner

**Location:** `_lym/plan/07-test-tier-check.md:58`

**Issue:** `internal/loomcli/validate.go` ends its Long help list with "Every mode but --require-approved also runs rename-tool-missing and pattern-changes-mismatch, which read language.yaml and the decision record."
Card 7 says the help "names it among the checks every mode but `--require-approved` runs", and cards 8 and 9 say only "names it" and "names `doc-go-unlisted`".
None says which sentence, and appending a name to this one leaves its trailing clause false: `test-tier-unstated` reads `gate.yaml`, `seam-test-unlisted` reads test files, `doc-go-unlisted` reads the tree.
Three cards editing one sentence with no stated owner is how the later card overwrites or contradicts the earlier one.
Typing is not the defect: card 7 and card 8 change `validatePlanCmd`'s code, and card 9 adds a name to the sentence the first two wrote, so `Edit` fits all three and no `Extends` is mistyped.

## Rejected

- Edit versus Extends on `internal/loomcli#loomCLI.validatePlanCmd` across cards 7, 8 and 9: all three type it `Edit`, and each changes existing help content, so no ref is an `Extends` on content the card rewrites.
- Matching of a base name that is the tail of a longer path: card 7 states the token rule and the `internal/x/a_test.go` against `internal/y/a_test.go` row pins it.
- The no-tier-line, several-tags, untagged-plus-tag and two-files-on-one-line states each have a failing row whose Detail names a fix and a row showing that fix passing, except as F1 to F3 record.
- Card 8's and card 9's own `internal/shedrecipe/doc.go` coverage: the D8 check is not in force while this plan is gated, and the doc names nothing the cards change.
