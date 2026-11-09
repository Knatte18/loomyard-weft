```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. I checked what each card's change forces outside the members the cards list.

- **MEDIUM — D8 (card 4): a user-visible refusal still promises the green-end resume.**
  - Evidence: `internal/landingshed/finalize_owner.go:130`, the `runOwner == ""` branch, says: `way forward: adopt it here with "%s", whose green adopt check resumes the halted run`.
  - Card 4 removes `resumeHaltedChildren`, so after it this refusal names a way forward that no longer happens.
  - The text is also the owner-halt reason that `runReports` shows through `haltReason` (card 5).
  - No card lists the enclosing member of `internal/landingshed/finalize_owner.go` or the package's doc.
  - The fix: card 4, or card 5 beside `ownerHaltResume`, rewords it to adopt here and then `lym tender resume` / `lym loom resume`.
  - A grep finds no test asserting the phrase.
- **LOW — D8: other docs still say the green end starts runs.**
  - `internal/beamcli/cli.go:780` (the `lym beam create` help text) and `internal/beamcli/doc.go:74` both say "only this verb, a hand pass or the green end of the adopt check that `lym adopt` starts starts or resumes its run".
  - `internal/battenshed/doc.go:98` says "The green end of the adopt check and `lym beam land` start a run's first pass detached".
  - `docs/overview.md:787` says "the subject's green adopt check lifts the holds."
  - Card 9 touches beamcli's `doc.go` and help text, but only to add the adopt section.
  - Neither these lines nor `battenshed/doc.go` nor `docs/overview.md` is listed on any card.
- **LOW — D5 (card 1): `pauserecord`'s `doc.go` and the `HoldCaller` comment need rewriting, not just extending.**
  - `internal/pauserecord/doc.go:22` says "Such an entry holds until its subject's adopt check is green".
  - `internal/pauserecord/record.go:26` says "Only the green adopt check of the entry's subject lifts it by itself."
  - Card 1 lists both, so they are covered. Its Intent says the doc changes but does not quote line 22. Flagged so the doc rewrite also covers the new `AdoptCaller` sentence beside line 21's `HoldCaller` text.
- **LOW — card 6: the file comment of `holds.go` goes stale.**
  - `internal/adoptcli/holds.go:1-3` reads "keeps the runs that an adopt holds until its subject's check… gives the pause-record entries that gate a held run and chooses which slugs to hold".
  - After card 6 that file keeps only the tries file (`readTries`, `writeTries`, `triesFile`, `triesFileName`).
  - No card lists the file comment, and `holds.go` cannot be named beside those member glyphs. The fix is for card 6's Intent to say the file comment is rewritten, or that the tries members move into another file.
- **LOW — card 5: one existing row still sets a deleted constant.**
  - `internal/adoptcli/runreport_test.go:82` sets `w.checkStates["alpha"] = checkStateEnded`.
  - `TestRunReports` is listed, so the edit is covered. Card 5's Intent should say that row is deleted with its state, not rewritten.
- **LOW — card 7: `cli.go`'s `Long` help text is stale between cards 7 and 10.**
  - `internal/adoptcli/cli.go:38-45`, in `Command`'s `Long`, still describes the hold and a green check that "lifts its holds and starts its runs".
  - Card 8 rewrites `Long`. Card 6 already changes the step order and the hold but does not list `Command`.
  - So after card 6 the help describes the old order until card 8. This is consistent if card 8 lands, but card 6's Intent should name it.

Extra tool calls (my context lacked each):
1. A repo-wide grep for `holds.json`, `adopt-hold` and green-end prose, plus `lym quarry resolve` on the cards' existing glyphs: I had not searched outside `adoptcli`, `pauserecord` and `beamcli`. The listed glyphs resolved.
2. `lym quarry resolve` output, `finalize_owner.go:120-135`, `beamcli/cli.go:776-782`, `beamcli/doc.go:72-76` and `battenshed/doc.go:96-100`: to read the stale sites the first grep found.
3. `TestCommand_RunsFromABeam`, the users of `checkStateNone`/`checkStateEnded`, batten cancel's lift filter and the stencils: to check card 8's test impact (validation still refuses before any spawn, so it holds) and other users of the deleted constants.
4. The webster stencil and markdown docs: to check stencil and skill prose about the adopt hold.
5. `docs/overview.md` line 787: to read the stale doc line.
