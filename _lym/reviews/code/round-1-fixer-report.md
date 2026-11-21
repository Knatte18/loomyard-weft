---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

## Changes

- F1 (`77b8828e`): `ErrRemoteLeftover` carries `originRef`, set to `ns.Branch(weftBranch)` where `probeLiveLocalWeft` builds the weft-kind error.
  The weft message's `push --delete` names that full ref; the warp message is unchanged.
  `remoteleftover_test.go` pins the message for a namespaced weft, an identity weft and a warp leftover.
  The one-repository integration row the review asked for was not added: the weft-divergence setup (`pushCommitToOrigin`) clones a branch name, which a namespaced origin does not hold.
  The unit rows cover the namespaced text, and the integration row below covers the probe setting the field.
- F2 (`444a30e4`): `failedRemoteSteps` names the records-side tag push as `refs/tags/<tag>:<ns.Tag(tag)>`, and omits it when the layout read fails.
  The code-store command is unchanged.
  `remoteteardown_namespace_integration_test.go` covers a namespaced and a default store.
  The existing `TestNoticeRemoteTeardown` rows still hold by substring.
- F3 (`c9e5965f`): the upstream-config failure in `pushCloneSeed` names the way forward: join with a plain `lym fabric clone <url>`, or set the board branch's `remote` and `merge` by hand.
- F4 (`11628bb1`): `TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine` has the namespaced tag rows, retryable under the namespace and not under identity.
- F5 (`bbee46b5`): the `doc.go` bullet leads with `goGit` resolving the common dir itself; the sentence after it was dropped as redundant.
- Self-check follow-ups, same defect class as F1 and F2:
  - `64aa6f07`: `TestAdd_LiveLocalWeftAgainstOrigin` asserts the diverged-weft refusal holds `push origin --delete refs/heads/<weft branch>`, so a probe that drops `originRef` fails it.
  - `fa896695`: `leftoverArchiveWayForward` in `internal/battencli/readoptstop.go` pushed the archive tag with no destination and deleted by the short name under a `refs/heads/` lease.
    It now names both through the new `fabricengine.OriginRefNamesAt`.
    The `checkUnlandedRemoteTip` refusal in `destroy.go` prints its by-hand delete with the full origin ref.
    `owner_readopt_test.go` rows and `recordslayout_refnames_integration_test.go` follow.

- Gate repair: three comment lines the round added were over-long or fixed-column wrapped.
  They are now one sentence per line:
  the `originRef` field comment in `remoteleftover.go`, the `failedRemoteSteps` doc in `remoteteardown.go`, and the header of `remoteteardown_namespace_integration_test.go`.
  No code changed.

- Gate reach notes: the second gate listed the tests that reach each changed or new symbol and named no symbol without one.
  The notes ask for tests only where reach is missing, so nothing was added and no code changed.

Gate: `lym gate test ./internal/fabricengine ./internal/battencli ./internal/gitrepo` at tiers 0 and 1 passed after the last change.

## Self-check

- fix-code-correctness, LOW: the `Error()` unit test sets `originRef` by hand, so a probe that stopped setting it passes — fixed, `64aa6f07` adds the integration assertion.
- fix-code-correctness, LOW: an externally built weft-kind `ErrRemoteLeftover` prints an empty ref — disputed, premise holds but no fix: `battencli` constructs them only in tests that never call `Error()`, and production sets the field at the one weft construction.
  Recorded here as a note, not a defect.
- fix-consistency, MEDIUM: `leftoverArchiveWayForward` twin — fixed, `fa896695`.
- fix-consistency, LOW: zero-`kind` weft error without `originRef` — same as the code-correctness item above, merged.
- fix-affected-set, MEDIUM: `leftoverArchiveWayForward` — duplicate of the fix-consistency finding, fixed, `fa896695`.
- fix-affected-set, LOW: discard-path tag push at `remoteteardown.go:186` is a code-store command — checked, no change.
- fix-affected-set, LOW: `destroy.go:1096` short-name delete hint — fixed, `fa896695`.
- fix-proposal, MEDIUM: `leftoverArchiveWayForward` — duplicate, fixed, `fa896695`.
- fix-proposal, LOW: zero-`kind` weft error — merged with the item above.
- fix-proposal, LOW: F3's `lym fabric clone <url>` may meet the hub the failed seed left — not changed: the message offers the hand-set keys as the alternative, and the review's own fix key proposed this text.

## Disputed

None.
The zero-`kind` self-check note above is a non-change with its reason, not a disputed review finding.

## Deferred

None.

## Rulings

None.
