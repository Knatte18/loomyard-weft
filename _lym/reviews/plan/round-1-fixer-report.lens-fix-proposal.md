```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out (F1, F2, F4, F5, F6, plus F3, whose exception is essentially the reviewer's). One proposal has a defect and one leaves an ambiguity a reader would notice.

**MEDIUM: F4 puts the `unpushed` error in the checked-locally detail, so a hub checked against origin is reported as "checked locally only"**
- Changed text, card 4 step 4: "an `unpushed` error is recorded in that hub's checked-locally detail and reads as not ahead, so the hub is still checked and swept".
- In the same step, the sibling text says that detail *marks* a hub: "an error or a non-empty skip marks the hub `checked locally only`, with its detail, and the check goes on against its local Bolt."
- Concrete case: hub X's `unpushed` read fails, but its `pull` succeeds with an empty skip. The collision check then ran against an up-to-date board. Yet the proposal puts the error in the field that reports X as checked locally only, so the output states something false about how X was checked.
- The `hubOutcome` field list in card 4 ("hub path, checked-locally detail, outcome, slugs, lost entries, error, way forward") has no other place for this note. The new `UnpushedReadErrorIsNotedAndSwept` row ("its outcome carries the error") doesn't say which field carries it.
- Suggested fix: give the unpushed read error its own note in the outcome, or carry it in `error` without marking the hub `failed`. The card can then state that a successful pull still reads as checked against origin.

**LOW: F3's exception still reads as if a replace sweep reports every beam owned elsewhere in `Lost`**
- Changed text, card 1 Intent: the `SetOwners` sentence "Without req.Takeover, an entry whose effective owner is another machine is neither written nor claimed" gains "unless req.Replace names its recorded owner". The `Takeover` field doc becomes "...left alone and reported in OwnerWrite.Lost, unless Replace names its recorded owner."
- Evidence: `ownerset.go:72-73` continues "...neither written nor claimed, / and it is reported in Lost." `writeOwners` adds to `lost` only for requested slugs or the prime (`ownerset.go:179`, `:195`).
- Card 1 says the replace selection "writes no other owner", so a beam owned by `far` is never selected and never reported in `Lost`. With only an "unless Replace names it" exception, a reader infers that a replace write reports `far`'s beam as lost.
- Suggested fix: say the sentence covers the entries the request names or selects.

The other proposals hold:
- **F2:** a rejected push replays the git commits and doesn't re-run `writeOwners` (`PushRecorded`, `fabricengine/bolt.go:131-147`). `written` is computed once, so replace-selected slugs reach the same re-read that `CompetingWriteLosesToOrigins` covers. Deleting the subtest is safe.
- **F1:** `hubOfDir` runs offline: `gitWorktreeRoot` uses an in-process `dotgit.FindRoot` (`lymcwd.go:152-162`), so the tier-0 cwd rows can run it. The concrete rows the fixer added work out from the merge rule in step 5, and `BAD` fails `CheckName` (it only accepts lower-case). Checked: `NoFileTakesTheHostnameNameAsOld` gives `["osl-1046","osl46"]`, `OwnFormerNamePassesTheCollisionCheck` gives `["neve","box","osl46"]`, and `EmptyEntryDropIsReported` gives `["box","osl46"]` with one entry dropped.
- **F5:** `SetOwners` errors carry their way forward in the text (`ownerset.go:278`, `:288`), so the outcome text plus the re-run field works.
- **F6:** there are four per-hub seams, and naming them fixes the count.

Tool calls:
1. Read `ownerset.go:95-140` and `:165-215` to check how `written` and the selection behave on push and replay. My context only had the re-read loop.
2. Grepped `pushOwnerCommit` (empty result), to see whether a retry re-runs `writeOwners`.
3. Read `pushOwnerCommit` and `Bolt.PushRecorded` to confirm a replay is a commit replay. My context lacked both.
4. Read `lymcwd.go:140-165` and `machinename.CheckName` to check that `hubOfDir` runs offline and that `BAD` is invalid. My context lacked both.
