```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changed text agrees with sibling cards, decisions, rows and cross-references (lens fix-consistency).

**[MEDIUM] Card 10's new removal order was applied to `accept` but not to its twin, `recheckStanding`**
- Changed text: `_lym/plan/10-beam-await-entry.md`, the new bullet "`awaitRun.accept` removes the await marker first, then the dobby-stop marker, then the land request … no crash leaves the await marker standing with the land request gone."
- Summary: the same card's `recheckStanding` bullet still says "then Done after removing the dobby-stop and await markers", which is the old order (dobby-stop first). That reopens F6's stall on the path card 10 adds.
- Evidence:
  - `recheckStanding` runs with no land request. It stops the dobby in `landChecks` and removes the dobby-stop marker before the await marker.
  - Take a crash between those two removals, then a resume from a halted status. Card 10's own line 46 says the resume rewrites the board to Beam-Await.
  - The entry then sees no hold on the board, the await marker still standing and no land request. Neither detection fires, and the dobby stays stopped.
  - The new bullet's "no crash leaves the await marker standing" is therefore false for card 10's own re-check.

**[LOW] Card 3: the `Check` parenthetical contradicts the new `missing` rule**
- Changed text: `_lym/plan/03-pr-merge-state.md`, the `RequiredChecks` bullet. The added sentence says "a context is `missing` only when neither a check run nor a commit status carries it".
- Summary: the same bullet still says `Check`'s Conclusion is "`missing` when no run of it exists at head", which was not updated.
- Evidence: in that bullet, "`Check` holds `Name` and `Conclusion` (empty while the check has not finished, `missing` when no run of it exists at head)" sits just before the added sentence.

**[LOW] Card 14: `pullrequest.Open` cannot be passed directly as `open`**
- Changed text: `_lym/plan/14-beam-pr-status-close.md`, "`PullRequestMode` calling `parentRuleRequired` … and `pullrequest.Open` as `open`".
- Summary: `open`'s type is `func(originURL string) (pullrequest.API, error)`, but `pullrequest.Open` returns `(*Client, error)`. Go does not convert between those function types, so the step as worded needs a wrapper.
- Evidence: the new Create glyph's signature, against `internal/pullrequest/client.go:113` (`func Open(originURL string) (*Client, error)`).

**[LOW] Card 15 names `parentRuleRequired` without listing it under Uses**
- Changed text: `_lym/plan/15-beam-ready-refusals.md`, refusal 5, "(card 14's `parentRuleRequired`, …)".
- Summary: card 15's Uses lacks `plan:internal/beamcli#parentRuleRequired`. Card 14's Uses did gain the cross-card symbols its fix names (`OriginMissing`, `pullrequest#Open`).
- Evidence: card 15's **Uses** list as edited, against card 14's.

Siblings checked that agree with the change set:
- Card 12's every-state-but-done rule matches card 15's refusal 1 ("running or halted").
- Cards 5 and 7 agree on the slot docs.
- Card 13's prOpen refusal of an empty beam, naming `lym beam ready`, now has a way forward through card 15's no-pass.
- Card 18 is unaffected.
- The overview's `PATTERN-pull-request-mode` line still holds, since the decision stays in `ResolvePullRequest`.

Tool calls:
- grep of `00-overview.md` and cards 13, 15, 18 and 9 for twin terms: my context did not hold the overview or those cards' surroundings.
- `wc` and grep of the overview, plus a listing of `_lym` and a search for the criteria files: to find where D3, D5 and D6 criteria live, which my context lacked.
- grep of `_lym/discussion/decision-record.md` for the criteria the fix touches (D5's halted run, D6's required checks, D11's accept order): my context did not hold the decision record.
