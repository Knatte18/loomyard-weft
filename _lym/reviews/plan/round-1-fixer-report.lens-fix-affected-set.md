```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check. For each finding whose premise is a set, I reran its search and checked that the fix covers every member.

**Findings left unchecked:**
- F3, the functions card 9's deletions leave unreachable. A full reachability search does not fit in the tool-call cap. I checked only that `parentWarning` keeps a caller under the fix (`takeOverRuns`/step 2) and that `workerOutName` is already in card 9's Delete list.

**Sets rerun:**
- **F4, the importers of `internal/adoptrecord`.** No gap. Every importer outside the package is covered: beamcli doc/adoptsection/cli and their tests are in card 6 or 5; orchcli `watch.go` and `underadopt_integration_test.go` are in card 5; the `adoptcli` files and tests are in card 9 (`stepsThrough` deleted; `world`, `TestAdopt`, `TestAdoptIntegration_SecondCloneTakesOver` edited).
- **F7, one-subject `lym adopt &lt;subject&gt;` ways forward in the adopt check, fix loop and green paths.** No gap. `fixloop.go:234` and `:250` sit in `failedEnd`, which card 8 edits, and its tests now assert `Rerun`. `green.go:37` goes with the count reset. `check.go:418/499/506` are covered by the `resolveSubjectWorktree` and `checkCmd` edits, and `checkview.go:97` by the `checkView` edit.
- **F10, mentions of "adopt worker".** No gap. `runproc/end.go:88`, `runproc/doc.go:62` and `lymdirs/dirs.go:41` are all in card 10 (formerly 11). The `adoptcli` ones go with card 9.

**Finding:**

- **MEDIUM.** Card 9, Intent step 2, the added sentence "Each moved run whose parent another machine owns carries `parentWarning`'s line in its outcome's `Warning`…", together with the unchanged claim at the top of card 9 that "Every way forward the steps word names the re-run over the subjects of the hold".
  - Summary: keeping `parentWarning` keeps a way forward, `lym adopt &lt;parent subject&gt;`, that card 2 case 3 refuses while this machine's hold stands. So the claim is false for the very line the fix kept.
  - Evidence:
    - `internal/adoptcli/adopt.go:691` returns "…so the run halts at Finalize here: `lym adopt %s` takes the parent over", where the subject is the parent's subject.
    - Card 2 case 3 refuses this machine's hold "over any other set, overlapping included", naming `lym adopt &lt;hold's subjects&gt;`.
    - The parent's subject is not among the hold's subjects, so the warning's command is refused until the hold clears.
    - The same applies to `heldBeamWayForward` at `adopt.go:630`, which the fix did not touch.
  - Ways out: reword the warning to name the parent's adopt only once the hold clears, or name the hold's subjects plus the parent's subject.

Tool calls beyond context:
1. Grep for one-subject `lym adopt` wording across `internal`, to rerun F7's search, which my context covered only for the sites the review named.
2. Grep for `adoptrecord` importers and "adopt worker" mentions, to rerun F4's and F10's searches.
3. Cards 5 and 6 target lists, plus the `adoptrecord` uses in each importing test file, since cards 5 and 6 and those tests were not in my context.
4. The functions owning `fixloop.go:234/250` and `adopt.go:618–691`, plus card 9's `parentWarning` lines, to attribute the remaining lines to the cards that edit them.
