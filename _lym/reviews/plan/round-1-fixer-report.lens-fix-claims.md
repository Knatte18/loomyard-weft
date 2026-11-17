```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked that every sentence the fix added or rewrote holds against the code at HEAD.

**MEDIUM: card 9, step 2's new sentence "Each moved run whose parent another machine owns carries `parentWarning`'s line in its outcome's `Warning`", and the kept row `ParentTakenByARaceCarriesTheFinalizeWarning`**
- **Summary:** The kept row asserts a pause field that card 9 retires and uses a race seam the foreground flow no longer has. The warning's way forward also contradicts card 9's hold-re-run claim.
- **The row cannot be kept as written.** At `internal/adoptcli/adopt_test.go:721-734` it drives `w.race` (the owners change between the owner read and the write) and asserts `alpha.AdoptPaused`. Card 9 drops the pause fields from `slugOutcome` and retires the adopt pause.
- **Wrong scope.** Today `parentWarning` runs in `resumeSlug` (`adopt.go:505`) for every settled entry, adopted beams included, not only for moved runs.
- **Wrong way forward.** `parentWarning` words it as `` `lym adopt &lt;parent subject&gt;` takes the parent over `` (`adopt.go:692`). Card 2 case 3 refuses that re-run as another set while this machine's hold stands. Card 9 says "Every way forward the steps word names the re-run over the subjects of the hold".

**LOW: card 8, "`TestGreenEnd` (tier 0) … asserts the board-read line names `Rerun`"**
- **Summary:** No `TestGreenEnd` row reaches the board-read line, so the assertion needs a new row that the card does not name.
- **Evidence:** the fixture's `Tasks` always returns `f.tasks, nil` (`internal/adoptcli/green_test.go:48`). `removeFixTask`'s board-read line (`green.go:49`) is reached only when `Tasks` errors.

**Claims checked that hold:**
- `FabricReplay` only logs a refusal (`bolt.go:393`), and the push then reads level (`ownerset.go:356-376`).
- `ParseOwners` reads empty input as `Owners{}` and decodes `machines` and `holds`.
- `OwnerArchivePair`'s slug for the prime is `root`.
- fronttask starts the action detached and writes its output to a file without reading it or its exit code (`hub.go:111-165`). The hand step appears in the start-error way forward (`fronttask.go:318`).
- `EndSubject`'s doc names the adopt worker (`end.go:88`).
- holdwrite.go's header says WriteHold is the only writer.
- On card 9, `Kept` is false only on a new hold (which has no fix task) and on a take-hold rewrite.
- Card 9 is the first card that targets `cmd/lym`. Card 4 only mentions it, with no edit.

Tool calls:
- Read `adopt.go` 490-510 and 678-700, `adopt_test.go` 718-735 and `green_test.go`. My context lacked `parentWarning`'s call site, the kept row's body and the green fixture's `Tasks`.
- Grepped cards 1-8 for `cmd/lym` to check the Shared Decision's claim that card 9 is the first card to target it; my context did not show cards 4-7 in full.
