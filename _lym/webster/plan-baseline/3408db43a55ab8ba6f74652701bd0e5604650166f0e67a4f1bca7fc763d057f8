# Card 6 — prime-records-branch-key

**Edit:**
- `internal/fabricengine#primaryWeftBranch`
- `internal/fabricengine_test#TestCleanup_ProtectsPrimaryWeftBranchAfterCheckout`
- `internal/fabricengine_test#TestCleanup_RefusesWhenPrimaryWeftBranchIsUndeterminable`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#originNamespaceAt`
- `internal/fabricengine#PrimeRecordsBranchKey`
- `plan:internal/hubforge#NewOneRepoHub`
- `internal/fabricengine#RecordsRepoRoot`
- `internal/fabricengine#Topology.Checkout`

**Intent:**
Make `primaryWeftBranch` read the prime's records branch from `lym.primeRecordsBranch` in a one-repository hub (D1),
so the orphan sweep and `resolveManagedBranch`, its callers, never treat it as an orphan.

`primaryWeftBranch` reads `originNamespaceAt` of the records store (`RecordsRepoRoot(l)`):
- identity, a two-repository hub: the board's branch plus the weft suffix, as today;
- the one-repository namespace: `git config --get lym.primeRecordsBranch` in the records store.
  The board is on `main` whatever the code default branch is, so its branch names nothing here.
  An unset key fails closed, as an unreadable board branch does, with the refusal
  "the records store at <dir> has no lym.primeRecordsBranch; refusing to enumerate orphan weft branches — set it with `git -C <dir> config lym.primeRecordsBranch` to <prime-branch>-weft".
  A failed layout or config read is refused the same way, naming the error.
`lym fabric checkout` writes no such key, so a checkout that moves the prime cannot move the prime's records branch.
`cleanup.go`'s file header says the primary is "the default branch plus "-weft""; this card rewrites that sentence to name both sources.
The header is no member's span, and `redundant-file-target` refuses the file glyph beside `primaryWeftBranch`, so the sentence is named here instead.

Tests:
- `TestCleanup_ProtectsPrimaryWeftBranchAfterCheckout` becomes a table over two builders, its body the shared row:
  - the existing row, over `CopyHub`, keeps its stand-in for the checkout;
  - a one-repository row over `NewOneRepoHub(OneRepoOptions{DefaultBranch: "master"})` moves the prime off `master` with `Topology.Checkout`, the verb behind `lym fabric checkout`;
    the orphan sweep then still lists and deletes nothing for master-weft, and the branch stays.
    Today's code protects main-weft there, so the row fails on it.
- `TestCleanup_RefusesWhenPrimaryWeftBranchIsUndeterminable` gains rows over a one-repository hub:
  - the records store's key unset: the sweep is refused, and the refusal names `git -C <records> config lym.primeRecordsBranch`;
  - the records store's `config` file overwritten with unparsable text: the sweep is refused, and the refusal names the read error.

`doc.go`'s orphan-sweep paragraph gains the one-repository source of the primary.

**ImpactSummary:** One function's source of truth changes for a one-repository hub only; its callers (`Topology.Cleanup`, `resolvePairWeftBranch`, `resolveManagedBranch`) are unchanged and a two-repository hub reads the board branch as before.
