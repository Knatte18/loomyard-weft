```yaml
lens: coverage
outcome: reported
```

Lens: coverage. I traced every decision, every acceptance criterion and every branch the cards introduce to the card and test row that reaches it. I made no further tool calls; I worked from what my context already held.

**MEDIUM: criterion 1 / Card 1. The empty `Replace` entry is never matched against an empty recorded owner.**
- `ReplaceSetSelectsAfterThePull` passes `Replace {"box", "", "old"}`, but by the time of the call no open beam has an empty owner: `pushFromOther` sets `beamer` to `box` and the prime is `box`.
- So the rule "an empty entry selects nothing, so an unowned beam is never taken" (`01-owner-replace-set.md` Intent) has no row that would fail if it broke.
- The fixture's only other unowned project is `grouping`, which is not a beam (`internal/boardengine#newOwnerFixture`).
- Fix: keep one unowned open beam in that subtest, for example leave `beamer` unowned and push `box` onto a new beam. Then assert that it stays unowned on origin.

**MEDIUM: Card 1. A competing write that keeps a selected entry is not reached on the replace path.**
- The Intent says the re-read reports in `OwnerWrite.Lost` a slug the replace set selected but a competing origin write kept.
- `CompetingWriteLosesToOrigins` (`internal/boardengine#TestSetOwners`) covers only a named slug.
- `LostEntryIsReportedPerEntry` in card 4 uses a fake `setOwners`, so it never exercises boardengine's re-read of a replace-selected slug.
- Fix: add a replace-set row with a `BeforePush` that pushes another owner onto a selected beam, and assert `Lost`.

**MEDIUM: Card 1. A slug that is both named in `Slugs` and selected by the replace set has no row.**
- The Intent says such a slug "is written once and listed once in `written`", and no row reaches that.

**MEDIUM: criterion 6 / D5. The verb never asserts what it reports about a repaired file.**
- D5 says "the output says so" when a file's invalid `name` is dropped and when an undecodable file's names are lost.
- Card 4 step 5 plans those notes.
- `TestNextFile` checks only the names it computes, and no `TestName_Set` row runs the verb over an invalid-name or undecodable file and asserts the note.
- `OwnFormerNamePassesTheCollisionCheck` uses a bad file but asserts only that the collision check passes.

**MEDIUM: criterion 8 / D5 + D6. The sweep of owners under the hostname rule's name is unreached at the verb level.**
- The criterion: over the hand-written `{"name":"osl46"}`, the verb records the hostname rule's name and sweeps owners recorded under it.
- `TestNextFile` `HandWrittenNameOnlyRecordsTheHost` covers only the names.
- No `TestName_Set` or `TestName_SweepsHubs` row runs the verb over that file and asserts that `Replace` holds the hostname rule's name, or that such an owner is rewritten.
- In `TestName_SweepsHubs` there is no file, so the hostname rule's name is the old current name, which is a different path.

**MEDIUM: Card 4. Four branches the Intent introduces have no row:**
- (a) A failed registry read refuses before any write, naming the record's path (step on `hubsToSweep`). The fake seam `hubs` can return an error, and no row is planned.
- (b) A `readOwners` error fails that hub while the check goes on (step 4).
- (c) A `pull` that returns an error, as opposed to a skip, marks the hub checked locally only (step 4). Only `PullSkipIsCheckedLocally` exists, and the tier-1 unreachable origin comes back from `Bolt.Pull` as a `fetch_failed` skip, not an error.
- (d) A cwd in a subdirectory of a hub worktree adds that hub through `lymcwd.ResolveWorktree` plus `preflight.BoardLymPresent`, and a cwd outside any hub adds none. Only `CwdHubIsSweptOnce` is planned.

**MEDIUM: Card 3. The show with no file and a failing `hostName` is unspecified and untested.**
- `readCurrent` takes the current name as the hostname rule's name when there is no file, and `""` when `hostName` errors.
- The card does not say what the show prints in that case (an error naming the hostname way forward, or an empty name), and no `TestName_Show` row reaches it.

**LOW: Card 4. The unreadable-file refusal (step 2, "a read error that is not a decode error") has no row.**
- It is exempt from reachability as a propagated I/O error, but D5's "a bad file" path includes it.
- A directory at the file's path reaches it cheaply, as `internal/machinename#TestResolveNames_UnreadableFileIsAnError` does.

**LOW: Card 2. `TestFromThisHost`'s error branch is unreachable on any machine with a hostname.**
- The row compares against the test's own `os.Hostname()`, which does not fail in practice.
- So "errors exactly when that does" pins only the success path.
- The error path is `nameFromHostname`'s, which `internal/machinename#TestResolveNames` `UnreadableHostnameNamesItsWayForward` already covers.
- So this is acceptable, but the row claims more than it reaches.

**LOW: Review criterion (D6 recoveries). Card 4 asserts the static `recoveries` text in every successful `TestName_Set` row.**
- The criterion is a `Review:` wording criterion, which the rubric's "Test economy" says a card plans no test for.
- Asserting only that the recoveries field is present would be the bound; matching the exact text tests prose.

**LOW: D5 collision refusal. No row asserts the refusal's way forward.**
- `CollisionIsRefusedNamingHubAndOwner` asserts the hub and the owner, but not the way forward: choose another name, or write `{"name": "<name>"}` by hand.
- `PATTERN-refusal-way-forward` asks that a test reach the way forward a refusal names.
