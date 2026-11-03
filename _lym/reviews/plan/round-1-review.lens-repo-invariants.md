```yaml
lens: repo-invariants
outcome: reported
```

Task: repo-invariants lens (test seams and tiers, doc text the cards falsify, card order) over cards 1–14.

**Per card, each invariant checked**
- Card 1: seams held, since no reflective test walks a gitrepo struct. Tiers held: `pushretry_test.go` is tier0 and `TestDeleteRemoteBranch` is tier1. Doc broken: the `pushretry.go` file header still says only two refusal reasons are retried, and `fabricengine/doc.go:642` lists two. `ErrLeaseRejected`, whose doc comment the Intent edits, is not a target. Order held.
- Card 2: seams and tiers held. The `push.go` file header lists the push surface without `PushBranch`; the list is incomplete, not false, so this is not a finding. Order held, since `pushPlainWithRetry`'s three call sites are all on the card.
- Card 3: seams, tiers (tier1) and order held. The `remoteleftover.go` header still holds.
- Card 4: seams, tiers and order held. The doc changes (the `AdoptLocal` rule) are listed.
- Card 5: tiers and order held. Doc partly broken: the Intent rewrites the `teardownlock.go` file header, but that file is not a target. Card 8 then adds the sweep as a holder without touching that header.
- Card 6: tiers and order held. Doc partly broken: `archiveWeftTip`'s doc comment is updated only in its retry sentence, though the function now pushes the branch and can push a second tag. The `PATTERN-fabric-git` background "first pushes an archive tag" now has a branch push ahead of it, and no card lists it.
- Card 7: tiers, order and seams held.
- Card 8: tiers and order held. Doc broken: `RemoteWarpBranchEntry.Reason` says it is empty for a candidate, and the card now writes a lock reason on a kept candidate. The `teardownlock.go` header and the holder text of `TeardownLockPath` omit the sweep.
- Card 9: tier1, seams and order held.
- Card 10: **tiers broken**, because tier0 test rows spawn git. **Doc and refusal text broken**: `taskWorktreeLocation` and `presentTaskWorktreePath` still say batten never re-creates a pair. Order held.
- Card 11: seams, tiers and order held.
- Card 12: tier0 fakes only, so tiers held. Order and seams held.
- Card 13: tier0 tests held. Import order held: battenrecipe's dependencies include neither pairteardown, tendercli nor battencli, so no cycle.
- Card 14: nothing to check.
- Guards: no new result type, so the destructive and mutation-record guards hold. The remote-refs fetch guard holds because the fetches go through `FetchWithArgs`. The raw-git guard scans only websterengine.

**Findings**

1. MEDIUM, design, card 10 (`TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal`). The new `taskWorktreeGone` rows sit in a tier0 test, but every row except "no path" spawns git.
   - The plain-file, empty-directory and resolving-worktree rows all call `lymcwd.ResolveWorktree`, which runs git (`internal/battencli/wire.go:235`).
   - The resolving-worktree row also needs a real worktree from gitkit or hubforge. Both are barred in a tier0 file, so the card's own tierpurity gate fails.
   - The test is tier0 (`internal/battencli/wire_test.go:689-690`).
   - Fix: move those rows to a tier1 test, such as `TestBattenIntegration_GoneTaskWorktree`, or a new tier1 file.
2. MEDIUM, consistency, card 10. The doc and the operator-facing refusal of `taskWorktreeLocation` and `presentTaskWorktreePath` become false once the pre-run re-adopts a gone pair.
   - Evidence: `internal/battencli/wire.go:203-209` and `:224`, which say "batten does not recreate a pair from its branch, and no \"lym fabric\" command currently does either" and send the operator to restore the pair by hand or delete the run directory.
   - Neither symbol is a target of any card. `TestTaskWorktree_AbsentPairIsAnAnswerAndANamedRefusal`, which card 10 edits, pins "Resolve it by hand" (`wire_test.go:707`).
   - Fix: list `internal/battencli#presentTaskWorktreePath` and `internal/battencli#taskWorktreeLocation` on card 10, and reword the text to name `lym batten run &lt;slug&gt;`, which re-adopts the pair.
3. LOW, consistency, card 1. The `pushretry.go` file header says a push is repeated "only when the server refuses the pushed ref with a bare \"(failed)\" or a remote \"cannot lock ref\"" (`internal/gitrepo/pushretry.go:2`).
   - The card adds a third reason. It edits the doc comments of `retryablePushRefusal` and `PushRebaseFree` and the paragraph in gitrepo's `doc.go`, but not this header.
   - `internal/fabricengine/doc.go:642` makes the same two-reason claim about every plain fabric push.
4. LOW, consistency, card 1. The Intent says "Update `ErrLeaseRejected`'s doc comment to name both callers", but `internal/gitrepo#ErrLeaseRejected` is not among the card's Edit targets (`internal/gitrepo/push.go:20`). Fix: add that member glyph.
5. LOW, consistency, cards 5 and 8. The `teardownlock.go` file header describes the lock as excluding Remove against the pair's detached pushes (`teardownlock.go:1-4`), and `TeardownLockPath`'s doc says the same (`:40`).
   - Card 5 names the header in its Intent but not in its targets.
   - Card 8 adds the cleanup sweep as a holder, but its Uses lists only `TeardownLockPath` and `lockPairTeardown`, and it updates neither text.
   - A file target beside member targets of the same file is refused by `redundant-file-target`. So the header goes on a card that holds no member of `teardownlock.go`, or is noted as carried by card 5's Intent.
6. LOW, consistency, card 8. `RemoteWarpBranchEntry.Reason` is documented as "empty for a candidate", and `Error` as the field set when apply tried a candidate (`internal/fabricengine/cleanupremotewarp.go:25-28`).
   - The card writes a held-lock `Reason` on a kept candidate, but lists `RemoteWarpBranchEntry` only under Uses, so that doc becomes false.
   - Fix: move it to Edit and update the field doc.
7. LOW, consistency, card 6. The doc comment of `archiveWeftTip` (`internal/fabricengine/archive.go:47-57`) says it tags the tip and pushes that tag, and returns one tag.
   - The card changes it to push the weft branch first, then push up to two tags, and return a `weftArchive`, but scopes the doc update to "the retry sentence".
8. NIT, consistency, card 6. The `PATTERN-fabric-git` background says every teardown "first pushes an `archive/&lt;slug&gt;/&lt;tip&gt;` tag" (`pattern/PATTERN-fabric-git.md:30`). The removal now pushes the weft branch before that tag, and no card lists the file.

Tool calls:
1. Grep of `//lym:seam` markers, pattern-enforcing tests and the tiers of the edited tests. The manifest had no tier lines or seam markers.
2. `wire_test.go` `TestTaskWorktree…`, the destructive-guard token lines and the remote-refs-lock guard header. Needed to judge the tier0 rows and the guard scope.
3. The destructive guard's banned-token list, the leaf-package list, tendercli's imports and the sites of the retry-reason docs. Needed for the import and doc checks the manifest lacked.
4. The push-classification guard's location and the roots of the raw-git guard. Needed to see whether a guard scans the new pushes.
5. The scope and tokens of the raw-git guard. Needed to rule out a raw-git break from gitrepo or battencli.
6. The mutation-record guard and the doc section that defines it. Needed to check that no card adds a result type that breaks it.
