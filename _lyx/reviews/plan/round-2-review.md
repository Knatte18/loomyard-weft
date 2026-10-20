---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/08-darn-landing-tree-scope.md:47
    summary: >-
      Card 8 prescribes that Merge-In, Finalize and Publish return darnTreeScope's editor way forward, but no named test reaches that error at any of the three sites.
    fix: >-
      In 08-darn-landing-tree-scope.md, extend the last paragraph (the one starting TestDarnTreeScope_ClassifiesAtEachCall also builds landingDeps) with a sentence.
      At Card 7's step where the hub darn.yaml holds ./tools under tree_scope, the test also asserts that VerifyCommand() returns an error naming ./tools, lyx config darn's editor and the clause then re-run the lyx command that refused, and that PublishVerifySkip() returns that same error.
      The existing sentence about the scoped string and the hub verify stays as it is, since this step answers neither.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-darn-gate-impact-scope.md:64
    summary: >-
      darnTreeScope returns every darn.yaml load error except ErrTreeScopeKey bare, and darnVerifySource.command and the Publish seam call it before VerifyCommand, so Merge-In, Finalize and Publish lose the way forward VerifyCommand appends today.
    fix: >-
      In 07-darn-gate-impact-scope.md, change the darnTreeScope(l) paragraph so the seam first calls darnengine.VerifyCommand over fabricengine.ConfigDirs(l) and returns its error unchanged.
      VerifyCommand already gives a malformed tree key the editor way forward and every other load error the verify way forward, so the clause that sends a LoadConfig error through WithEditorWayForward is dropped there.
      The seam then loads the tree keys through darnengine.LoadConfig, and the sentence that every other error is returned as is keeps only the merge-base and diff reads.
      Add one row to TestDarnTreeScope_ClassifiesAtEachCall with a hub darn.yaml that does not unmarshal, asserting the seam error names lyx config darn --set verify= and the clause then lyx loom resume.
      In 02-darn-tree-scope-config.md, remove darnTreeScope from the sentence that lists the callers of WithEditorWayForward.
      In 08-darn-landing-tree-scope.md, no change is needed beyond what F1 adds.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/08-darn-landing-tree-scope.md:32
    summary: >-
      Card 8 says a Publish seam error is returned the way the merge-commit read fails, but that read only logs a warning and never fails Publish.
    fix: >-
      In 08-darn-landing-tree-scope.md, replace the clause read the way the merge-commit read fails with the clause the way holdParent returns a parent-guard read error.
      The sentence then reads a seam error is Publish's returned error, the way holdParent returns a parent-guard read error, with nothing verified or pushed.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/00-overview.md:37
    summary: >-
      The verify block runs no test of internal/websterengine, whose card gate consumes the GuardScanError and FindGuardTests that Card 4 edits and says it leaves reading as before.
    fix: >-
      In 00-overview.md, append ./internal/websterengine to the end of the lyx gate test --tags integration line of the verify block, after ./internal/loomrecipe.
      Checked with lyx loom validate-plan --overlay on the overview with that one change, which reports no finding beyond the baseline.
---

### [MEDIUM] No test reaches the editor way forward at the landing sites

**Location:** `_lyx/plan/08-darn-landing-tree-scope.md:47`

**Issue:** Card 8's Intent states that Merge-In's and Finalize's "read verify command" error and Publish's seam error name `lyx config darn`'s editor, because `darnVerifySource.command` and `publishSkip` pass `darnTreeScope`'s error through.
`PATTERN-refusal-way-forward` requires a test that reaches each such refusal and asserts its message.
The only test Card 8 names for these sites asserts, at each of Card 7's steps, the scoped string or the hub `verify` and `PublishVerifySkip()` true or false.
Card 7's `./tools` step answers neither, so no assertion covers it.
The `TestPublishVerify_HardErrorsAreReturned` row feeds a stub seam error and pins only that its text comes back, so it does not reach the editor refusal either.
The seam-level step in Card 7 covers `darnTreeScope` alone, not the three call sites.

### [MEDIUM] darnTreeScope drops the way forward VerifyCommand gives other load errors

**Location:** `_lyx/plan/07-darn-gate-impact-scope.md:64`

**Issue:** Today `darnVerifySource.command` is `darnengine.VerifyCommand`, which appends `verifyWayForward` to every darn.yaml load error.
Card 8 makes `command` ask `darnTreeScope` first, and Card 8's `publishSkip` calls it alone at Publish.
`darnTreeScope` returns a `LoadConfig` error through `WithEditorWayForward`, which leaves every error not wrapping `ErrTreeScopeKey` unchanged.
So an unparsable darn.yaml, or any load error whose own text names no fix, such as `unmarshal darn config: ...` in `internal/darnengine/config.go`, now reaches Merge-In, Finalize and Publish with no way forward, where before it carried one.
That departs from `PATTERN-refusal-way-forward` for `internal/loomcli`.
The Darn-Write and Darn-Burler-Fix gates are not affected, because their step 1 reads the told command through `VerifyCommand` before the seam runs.
The fix routes the seam's load step through `VerifyCommand`, so the one function that already owns both clauses decides them.

### [LOW] A seam error is compared to a read that never fails

**Location:** `_lyx/plan/08-darn-landing-tree-scope.md:32`

**Issue:** The card says a seam error is Publish's returned error, "read the way the merge-commit read fails".
`takeMergeCommitFromRecord` in `internal/landingshed/publish.go` logs a warning for an unreadable record and carries on, so the reference describes the opposite of "returned, nothing verified or pushed".
An implementer following the comparison could log and proceed, which would run `publish_verify` or skip it over a diff that was never classified.
`holdParent` returns its read error from `Call` with nothing mutated, which is the behavior the sentence asks for.

### [LOW] The verify block leaves out the consumer of the guard-scan error Card 4 edits

**Location:** `_lyx/plan/00-overview.md:37`

**Issue:** Card 4 adds fields to `impactset.GuardScanError` and changes `guardNamesInFile`, and its Intent says webster's card gate reads as before.
`internal/websterengine/guardstep.go` calls `impactset.FindGuardTests` and `errors.As`es the `GuardScanError`, and `beginbatch_test.go` and `cardgate_test.go` pin that behavior against real marked files.
Those tests are the ones that would show the claim false, and the block lists neither the package nor any run that reaches it.
Every other package the block lists is one a card targets or `cmd/lyx`, so this is a consumer that is not targeted, and the rubric's coverage rule names targeted packages only; the finding rests on Card 4's own claim about webster.

## Rejected

- `docs/overview.md`, which Cards 5, 6 and 9 edit, is read by the `//lyx:guard` test `TestEnforcement_MarkdownLinks` in `internal/lyxcwd`, and the verify block does not run that package.
  Not a finding: `docs/` is no Go package a card targets, and the rubric's coverage rule names targeted packages.
- The editor save check's two way-forward branches in Card 3: removing the last tree while `tree_scope_verify` is non-empty names `lyx config darn --set tree_scope_verify=` first, and every other load refusal names the copy-out from the kept staging copy.
  Both match the decision record's D1 and are steps the operator completes from the prime, since every open of the editor rewrites the staging copy from the hub file and `--set tree_scope_verify=` loads with trees present.
- Package coverage of every Go package the cards target, including the packages holding the edited contract spec `shed-recipe-spec.md` (`contracts/specs`) and the darn writer stencil (`contracts/stencils`): all are in the verify block, and `contracts/recipes` holds no test of its own, its recipe being read by `loomrecipe` and `loomcli` tests that the block runs.
