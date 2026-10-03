---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/15-webster-verify-gate.md:Edit
    summary: card 15's fixer-commit walk needs a gitrepo.Repo for parentMergeRejection, but the card does not target gitwrap.go, so verifygate.go must call gitrepo.New and fail TestNoRawGitMutation_WebsterProductionSource
  - id: F2
    severity: LOW
    location: _lyx/plan/17-retire-integration-stage.md:Edit
    summary: no card removes the gitwrap.go helpers integrationfix.go was the last caller of (integrationFixHeadRefusal, and commitParentCount unless card 15 reuses it)
  - id: F3
    severity: LOW
    location: _lyx/plan/17-retire-integration-stage.md:Edit
    summary: cmd/lyx/rawgitmutation_test.go's header comment names FabricBisector, which card 17 deletes, and no card updates it
  - id: F4
    severity: LOW
    location: _lyx/plan/17-retire-integration-stage.md:Edit
    summary: fabriccli merge-in's webster in-flight warning and help text still promise an integration bisect over card SHAs after card 17 retires it
  - id: F5
    severity: NIT
    location: _lyx/plan/17-retire-integration-stage.md:Edit
    summary: comments naming the retired bisect survive in internal/verifyrun/run.go's package doc, recoverbatch.go's CardSHAs comment and state.go's CardSHAs field doc
  - id: F6
    severity: LOW
    location: _lyx/plan/17-retire-integration-stage.md:Intent
    summary: the raw CheckoutDetached/RestoreBranch primitives in gitrepo and fabricengine lose their only caller with card 17, and their docs keep naming the integration bisect as their one consumer
---

### [MEDIUM] Card 15's commit walk cannot live in `verifygate.go`

**Location:** `_lyx/plan/15-webster-verify-gate.md`, `**Edit:**` list and closure step 3.

**Issue:**
Step 3 judges each commit from the pre-fix head to HEAD with `gitwrap.go`'s `parentMergeRejection`, which takes a `*gitrepo.Repo`, and resolves the parent tips with `resolveParentTips`, which takes one too.
`gitrepo.New` is the only constructor, and `cmd/lyx/rawgitmutation_test.go`'s untagged `TestNoRawGitMutation_WebsterProductionSource` fails every websterengine production file containing `gitrepo.New(` except the allowlisted `gitwrap.go`.
Card 15 lists `gitwrap.go` only under `**Uses:**`, so the card as written leads the implementer to construct a repo in `verifygate.go` and fail its own per-card gate (`go test ./...`), or to edit `gitwrap.go` off-target.
The card also builds the report's fix-commit list "from the pre-fix head to HEAD", whose existing implementation (`fixCommitsSince`) lives in `integrationfix.go`, a file card 15 does not target and card 17 deletes.

**Fix:**
Card 15 adds `internal/websterengine/gitwrap.go` to its `**Edit:**` list and names the helpers it adds there:
one returning the first-parent commits from the pre-fix head to HEAD, oldest first (over `commitsBetween`), and one returning the first commit on that chain that is neither a non-merge commit nor a clean parent merge, with the rejection reason (over `CommitParents`, `resolveParentTips` and `parentMergeRejection`).
`verifygate.go` calls only those helpers and never constructs a repo.

### [LOW] `gitwrap.go` helpers left without a caller

**Location:** `_lyx/plan/17-retire-integration-stage.md`, `**Edit:**` list.

**Issue:**
Outside `gitwrap.go`, `integrationFixHeadRefusal`, `commitsBetween` and `commitParentCount` are used only by `integrationfix.go`, which card 17 deletes.
Card 17's intent is to remove what cards 15 and 16 left without a caller, yet it does not target `gitwrap.go`.
`integrationFixHeadRefusal` is dead after card 17 whatever card 15 does;
`commitParentCount` is dead unless card 15's walk reuses it.
Go does not reject an unused package-level function or variable, so nothing catches the leftovers.

**Fix:**
Card 17 targets `gitwrap.go` and deletes `integrationFixHeadRefusal` and every other helper card 15's walk leaves without a caller (with F1's fix, `commitParentCount`; `commitsBetween` stays as the walk's base).

### [LOW] Raw-git guard comment names a deleted interface

**Location:** `_lyx/plan/17-retire-integration-stage.md`, `**Edit:**` list.

**Issue:**
`cmd/lyx/rawgitmutation_test.go`'s header comment explains the token ban by the consumer code that calls `.CheckoutDetached(`/`.ResetHard(` on "the new FabricBisector/FabricResetter interfaces".
Card 17 deletes `FabricBisector` (`FabricResetter` is already gone), so the comment's example cites nothing that exists.
No card targets the file.

**Fix:**
Card 17 targets `cmd/lyx/rawgitmutation_test.go` and rewords the header comment's verb-name example so it names no retired interface.

### [LOW] merge-in still promises an integration bisect

**Location:** `_lyx/plan/17-retire-integration-stage.md`, `**Edit:**` list; `internal/fabriccli/merge_verbs.go` lines 30–32 and 140–143.

**Issue:**
`websterInFlightWarning`, emitted in the merge-in envelope while a webster run is in flight, and the merge-in help text both say "integration bisect over earlier card SHAs will run on pre-merge trees".
D4 retires the in-process bisect, and no card updates either text, so after card 17 merge-in tells Merriam and the operator about a mechanism that no longer exists.

**Fix:**
Card 17 targets `internal/fabriccli/merge_verbs.go` and drops the bisect clause from both texts, keeping the record-batch clause.

### [NIT] Stale bisect comments

**Location:** `_lyx/plan/17-retire-integration-stage.md`, `**Edit:**` list.

**Issue:**
`internal/verifyrun/run.go`'s package doc names "webster's integration bisect" as a consumer;
`internal/websterengine/recoverbatch.go`'s CardSHAs comment says it records them "so integration bisect has no gaps";
`internal/websterengine/state.go`'s `CardSHAs` doc calls the trail the "SHA-bisect anchor set".
After card 17 the consumers are `verifytree`, the per-card `**Verify:**` rerun and the card hint.
Card 17 targets `state.go` but its intent names only the `IntegrationFix` removal there, and no card targets `run.go` or names the `recoverbatch.go` comment.

**Fix:**
Card 17 targets `internal/verifyrun/run.go` and `internal/websterengine/recoverbatch.go` and names all three comments in its intent, each rewritten to name the card hint or the surviving consumers.

### [LOW] Detached-checkout primitives outlive their only caller

Found during job B, while grepping for the merge-in warning's other "integration bisect" mentions (F4); recorded here before any fix for it was made.

**Location:** `_lyx/plan/17-retire-integration-stage.md`, `**Intent:**`; `internal/gitrepo/gitrepo.go` (`CheckoutDetached`, `RestoreBranch`), `internal/fabricengine/warpforward.go`.

**Issue:**
D4 retires "every detached checkout of the live worktree".
The only production callers of `Fabric.CheckoutDetached`/`RestoreBranch` and `gitrepo.Repo.CheckoutDetached`/`RestoreBranch` are `integration.go`'s bisect and `triage.go`'s baseline, both removed by card 17.
`internal/gitrepo/doc.go` admits the pair as "the one admitted exception ... used solely by webster's integration bisect", and `internal/fabricengine/doc.go` keeps them unguarded as "raw primitives driven only by webster's integration bisect".
After card 17 those rationales are false, and two raw, merge-record-unguarded mutation primitives stay exported with no caller.
`CurrentBranch` keeps live callers (`fabricengine/weftgit.go`, `loomcli/landingbranches.go`) and is unaffected.

**Fix:**
A new card retires `CheckoutDetached` and `RestoreBranch` from `gitrepo` and `fabricengine` with their tests, drops them from `cmd/lyx/gitrepoboundary_test.go`'s pinned list, and rewrites both package docs' exception paragraphs.
