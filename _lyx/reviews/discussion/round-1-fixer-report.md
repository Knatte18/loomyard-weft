# Round 1 fixer report

Every finding is fixed; nothing is deferred.
`lyx loom validate-discussion` exits clean on the result.
Edits are in `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`.

## Changes

- F1 (BLOCKING): D3 now drops the prime's code side on every route that hands it to a push, `Fabric.Commit`'s detached spawn included, so `fabric commit`, `sync` and `lyx config`'s `sync` leave `main` alone.
  Scope In names the routes, the Plan chooses where the `Commit` route learns it runs in the prime, and the prime-name failure now fails the verb before it commits or pushes.
  Acceptance criteria gain `fabric commit` in the prime and in a pair.
- F2 (BLOCKING): D2's origin deletion now requires that the pre-flight probe found no branch on origin, not merely no `warpAdoptTip`, so a pre-existing branch at or behind `HEAD` that step 11 pushes forward is never deleted.
  Bound and an acceptance criterion pin the case.
- F3 (MEDIUM): the `PATTERN-mutation-record` constraint no longer sends push retries into the record; only the accepted push records, and the remote deletion records through its existing success-only entry.
- F4 (LOW): the same bullet now says the pattern's `mutations` and `partial` keys are untouched and that D3's and D4's fields sit outside that set; D4 calls the `remove` fields result keys.
- F5 (LOW): D1's Bound restated as the idempotence of a non-forced push of one commit to one ref.
  D2 resolves an exhausted step 11: the deletion runs once step 11 was attempted, leased to the pushed commit, so it refuses harmlessly when nothing landed and deletes the ref when it did.
- F6 (LOW): D1's classifier is one rule: a `! [remote rejected] <ref> -> <ref> (failed)` line for the pushed ref decides, and `remote:` lines neither qualify nor disqualify.
  An acceptance criterion pins the mixed-stderr case.
- F7 (LOW): acceptance criteria added for the prime-name failure, the code-side `push_spawned` absence in `sync`, the `code_worktree` key and "records sibling" wording, and the `hubgeom` lock-directory helper.
  D2's pattern edit now names `rollbackAdd`'s doc and a new line in `PATTERN-fabric-git.md`'s teardown section, not the archive-tag exception.
- F8 (LOW): the hard-cut assumption is narrowed to "nothing consumes them live", and the stranded-trace open risk states that the driver's rule escalates on an unknown `side`, so an operator removes the branch.
- F9 (NIT): the incident narrative in D1 and D2 Rationale, the support-log pointer in the first assumption, and the open risk restating D2's Bound are removed; the Bound's consequence moved into D2.
  The support log's interview, rejected alternatives and ledger now carry the evidence and the changed picks (Q4, Q6).

## Deferred

None.
