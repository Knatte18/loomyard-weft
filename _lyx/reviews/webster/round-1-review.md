---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/checkout.go:103
    summary: New and changed Go comments are hard-wrapped mid-sentence, or hold two sentences on one line, against PATTERN-comment-line-breaks
---

Range: `product.parent` is `main` and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (six commits, one per card).
`go build ./...`, `go vet -tags llm` over the card packages, `go test ./...` and `go test -tags integration` over the card packages all pass.

### [BLOCKING] Changed comments are wrapped mid-sentence instead of using semantic line breaks

**Location:** `internal/fabricengine/checkout.go:103`, `internal/fabricengine/checkout.go:137-146`, `internal/fabricengine/checkout.go:191`, `internal/fabricengine/mutation.go:65-67`, `internal/fabricengine/reconcile.go:5-8`, `internal/fabricengine/reconcile.go:41`, `internal/fabricengine/reconcile.go:526`, `internal/landingshed/finalize.go:164`, `internal/landingshed/finalize.go:218-221`, `internal/landingshed/finalize.go:261`
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line in every new or changed Go comment, with no column limit.
The sites above carry sentences that break at a column in the middle of a clause, for example `switchOrForkWeft`'s doc comment ("... the local" / "branch, else ...") and `Finalize.Call`'s Step 1c comment.
The `configChangeNotice` doc comment at `finalize.go:218-221` also puts two sentences on one line ("... wired. Finalize never carries ...").
These lines were added or edited by this diff, so the legacy-wrap exemption for untouched lines does not apply.
A violation of a listed constraint is BLOCKING.
**Fix:** Rewrite each listed comment (only the changed sentences) with one sentence per line, breaking further only at an independent-clause boundary, and split the two-sentence line in `configChangeNotice`'s doc comment.

## Focus verdicts

No other finding; each focus question is settled below.

- **Card 1, birth paths:** `createWeftWorktree`, `createWeftWorktreeDroppingRuns`, `adoptWeftWorktree` and Add's local-adopt branch each call `ensureWeftLockDirAt` before returning success, and each failure is wrapped with the weft path.
  Reconcile heals a missing `.weft/` through `restoreWeftLockDir` (records `KindDirCreated`, never touches `pr.Action`).
  `ReadOriginFor` refuses a record without it, naming the worktree and `lyx fabric reconcile`.
  The board weft worktree in `boardweft.go` is not a pair worktree and has no origin record, so it is outside the card.
  No bypass found.
- **Cards 2-4, resolver:** `checkout` (`switchOrForkWeft`), `reconcile` (`reconcileMissingWeft`) and `add` (live-pair step) all route through `resolveWeftBranch`; no caller forks from a bare `weftBranchExists`.
  Add's pre-flight probes are the read-only decision step, and Add's creation step then calls the resolver.
- **Mutations and rollback:** every new creation appends to the verb's record (`KindBranchCreated` in the resolver and for the adopted warp branch, `KindRepoAdvanced` for the fast-forward).
  Origin-tracked branch rollback goes through `deleteBranch` (`rollbackSwitch`, `rollbackAdd`, `deleteAdoptedWeftBranch`).
  Nothing pushes, force-pushes or deletes an origin branch in the new code.
- **Origin handling:** a missing `origin` remote skips the origin step, and an unreachable or failing origin returns an error naming `origin` (`remoteHeadTip`, the fetch and branch-creation wraps); no path forks silently.
- **Card 4 cases:** adopting origin weft and warp, fast-forwarding a behind local weft, and keeping an existing origin record match the card.
  A diverged local weft is refused with `leftoverDivergedAdoptedWeft` at pre-flight, and a local weft ahead of origin needs nothing (origin tip is an ancestor of local); both are the card's rule and not defects.
- **Cards 5-6:** `ReadConfigChanges` is read-only and carries nothing.
  `Finalize` queues one notice only on the `pushParent` Done return, `landingshed` names neither fabric side and derives no path, and the wiring runs through `loomcli`'s `landingDeps` closures.
- **Git access:** no new `//gitexec:raw` site; all new warp and weft git in `internal/fabricengine` goes through `gitexec.Run`.
- **Per-card mechanical check:** the plan's cards carry no Rename or Delete group beyond `leftoverUnarchivedWeft` (card 4); its removal leaves no remaining reference, as the build and vet show.
- **Test economy:** I found no new test that duplicates a named existing test, and no added behavior without a covering test.
- **Docs:** `doc.go` (fabricengine, hubgeom, landingshed) and the `lyx fabric add` help text land in the same commits as the behavior; no new cross-cutting invariant, so `PATTERN.md` and `docs/overview.md` need no change.
