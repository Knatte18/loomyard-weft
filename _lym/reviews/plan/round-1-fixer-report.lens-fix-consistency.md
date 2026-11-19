```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling cards, decisions and criteria it touches.

No findings.

The edits agree with their surroundings:
- **F1 and F10:** The empty-revision refusal in card 4 matches the reworded Shared Decisions bullet, card 3 ("`create` sends no revision") and D8 line 238 (a create carries no revision).
- **F3:** The new exit order in card 5's step 6 matches D11 at decision-record lines 334-336: stop taking new requests, release the lock, then wait for the requests in flight. `TestRun`'s drain row still holds.
- **F6:** With row 4 cut from card 2, the D8 criterion at decision-record line 508 is still covered by card 4's `TestServer` row "an edit carrying the `revision` of the entry a successful write answered is saved".
- **F2:** Card 7's Bound cites D6's bound (decision-record line 209) accurately. Card 9's Bound matches card 8's refusals.
- **F8 and F9:** Card 4 now extends `docs/overview.md` and comes before cards 7 and 9, which also touch that file, so the order holds. Card 6's Intent still says `doc.go` names the role, which fits moving `internal/runproc/doc.go` to Edit.

Further tool calls:
- One grep of `_lym/discussion/decision-record.md` for revision and exit-order text. My context held only that file's bound lines, not the D8 or D11 body and criteria lines.
