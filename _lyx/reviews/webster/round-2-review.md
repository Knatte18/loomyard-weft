---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/checkout.go:103
    summary: Changed comments in checkout.go, mutation.go, reconcile.go and finalize.go still wrap mid-clause, which PATTERN-comment-line-breaks forbids
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/remoteleftover.go:30
    summary: ErrRemoteLeftover doc splits a colon-introduced list of fragments across two lines, a mid-clause break
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/doc.go:494
    summary: Changed sentences keep an independent clause after a semicolon or a comma plus conjunction on one line instead of breaking there
---

Range: `status.json` gives `product.parent` = `main`, and there is no `_lyx/loom/rework/` directory, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
`go build ./...` and `go vet` over the touched packages pass.
The round-1 fix (`5926b160e`) re-broke the four named files only at sentence level: it moved sentence ends onto their own lines but kept wraps that fall inside a clause.
I read the plan's cards against the diff only for the code paths the comments sit on (`resolveWeftBranch`, `ReadConfigChanges`, the Finalize notice) and found no code defect.

### [BLOCKING] Mid-clause wraps in the four named files

**Location:** `internal/fabricengine/checkout.go:103-104`, `checkout.go:137-138`, `checkout.go:142-144`, `checkout.go:188-190`, `internal/fabricengine/mutation.go:65-67`, `internal/fabricengine/reconcile.go:5-7`, `internal/landingshed/finalize.go:218-219`.
**Issue:** `PATTERN-comment-line-breaks` allows a further break only at an independent-clause boundary, and each of these breaks falls inside a clause:
- `checkout.go:103-104`: the line ends at a comma before the fragment `also when the weft switch itself failed ...`.
- `checkout.go:137-138`: the colon is followed by the fragment list `the local branch, else ...`, which belongs on the line of the clause it completes.
- `checkout.go:142-144`: `and additionally records KindBranchCreated ...` is the second half of a compound predicate, and the colon after it is followed by the fragment `from the resolver when adopted ...`.
- `checkout.go:188-190`: the participle `cleaning up ...` and the fragment `with errors silently discarded.` are cut off from the clause they modify.
- `mutation.go:65-67`: the three items of the list after the colon are split over lines, each a fragment.
- `reconcile.go:5-7`: the compound predicate `recreates ..., re-points ..., adopts ..., and reports ...` is split at each predicate.
- `finalize.go:218-219`: `or "" when there is nothing to report ...` is a fragment, not a clause.

Clean in the named regions: `checkout.go:139-141`, `reconcile.go:8`, `reconcile.go:40`, `reconcile.go:524-527`, `finalize.go:164-166`, `finalize.go:220-221` as to wrapping, `finalize.go:261-262`.
**Fix:** Join each fragment onto the line of the clause it belongs to, and keep a break only where an independent clause starts (after a semicolon, or after a comma before a coordinating conjunction that opens a new clause).
For `reconcile.go` the sentence that begins at line 3 is changed, so reflow it whole: break after `valid paired topology:` and put the predicate list on one line.
For `checkout.go:137-144` break after `rec is Checkout's own recorder;` (already so) and keep `it records ... and additionally records KindBranchCreated for a created branch: from the resolver ... since `switch -c` creates it.` on one line.

### [BLOCKING] Colon-introduced fragment list split in the ErrRemoteLeftover doc

**Location:** `internal/fabricengine/remoteleftover.go:30-31`
**Issue:** `ErrRemoteLeftover is Add's refusal when a remote branch cannot be adopted or proven replaceable:` is followed on the next line by the fragments `a remote weft branch that diverges ..., or a remote warp branch ...`.
The second line is not an independent clause, so the break is mid-clause.
**Fix:** Put the fragment list on the same line as the colon.

### [BLOCKING] Independent clauses left on one line in changed sentences

**Location:** `internal/fabricengine/doc.go:494`, `doc.go:510`, `doc.go:514`, `internal/fabricengine/weftbranch.go:16`, `internal/fabricengine/configchanges.go:26`, `internal/fabricengine/reconcile.go:363`, `internal/fabriccli/adoptremote_integration_test.go:131`, `adoptremote_integration_test.go:132`, `adoptremote_integration_test.go:271`, `internal/landingshed/finalize_integration_test.go:106`, `internal/landingshed/finalize.go:220`, `internal/fabricengine/add.go:213`, `internal/fabricengine/doc.go:509`.
**Issue:** each line holds two independent clauses, joined by a semicolon or by a comma plus `so`/`and`, with no break between them.
Examples: `doc.go:494` (`...tip; archive tags are consulted ...`), `weftbranch.go:16` (`...is created; that returns exists=true ...`), `reconcile.go:363` (`It never changes pr.Action; a failure sets pr.Error.`), `finalize.go:220` (`..., so the notice tells the orchestrator ...`).
The same shape recurs in changed sentences elsewhere in the diff, for example `landingshed/doc.go:32`, `landingshed/deps.go:126` and `:135`, `fabricengine/add.go:75-77`, `:163`, `:288` and `fabricengine/doc.go:495-508`.
A sentence that is short and joins a compound predicate or a list is not a finding, and a semicolon inside parentheses (`checkout.go:189`) is exempt.
**Fix:** Break each such sentence at the semicolon, or after the comma before `so`/`and`/`but`, so the second clause starts its own line.
Sweep the whole diff's changed comment sentences with `git diff -U0 <merge-base>..HEAD -- '*.go'` filtered to added `//` lines containing `; ` or `, so `/`, and `/`, but `, and judge each against the independent-clause test; leave untouched legacy lines alone.
