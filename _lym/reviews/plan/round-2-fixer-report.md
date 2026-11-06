# Round 2 fixer-report

All four findings are fixed.
`lym loom validate-plan` answers `ok: true` after the last edit.

## Changes

- **F1, fixed as proposed.**
  In `05-hubs-verb.md`, `internal/creelcli/doc.go` and `docs/overview.md` moved from **Extends:** to **Edit:**.
  In `02-creel-config.md`, `pattern/PATTERN-config-layers.md` and `docs/overview.md` moved the same way.
  Both empty **Extends:** groups are gone.
- **F2, fixed as proposed.**
  The none refusal in `01-hub-registry.md` now reads `way forward: run lym creel inbox in that hub once`.
  `TestLookup`'s row is unchanged.
- **F3, fixed as proposed, with three additions and one choice of form.**
  In `08-outbound-worker.md`:
  - a neither-word answer now gives `bad-answer` with none of its text in the reason;
  - the neither-word row states what the notice gives at the hour;
  - the over-200-byte row names its input, a `hubFake` answering `refused` with a hostile `Reason`;
  - a new row covers a non-JSON line.

  Additions beyond the proposal:
  - the quoting sentence now names a `refused` answer's `Reason`, so the rule the row tests is stated;
  - the hostile `Reason` holds a multibyte rune across byte 200, since no other input reaches the rune-boundary cut;
  - the row says the refusal returns at once.

  In `07-receive-and-confirm.md`, `listenHubFake`'s `answer` now returns `(creelengine.Response, string)`, the string a raw line written verbatim with a line end.
  A second return value keeps the review's "no target is added", where a new answer type would have added one.
- **F4, fixed differently.**
  `05-hubs-verb.md` states that a failing `seam.registryDir` is propagated raw and gains a `TestCreelCLI` row.
  Not made: the proposed claim that the cause names `XDG_STATE_HOME` and `HOME` without qualification.
  Only `errMissingStateHome` (`internal/standalonestate/standalonestate.go:31`) names both;
  `errRelativeHome` (`:36`) reads `standalonestate: home directory must be absolute`.
  The card says "a missing home" instead.

## Self-check

- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings; no finding left unchecked.
- fix-consistency, LOW, card 1's none refusal departs from D1's "any `lym creel` verb" with no recorded departure: fixed with my own change.
  The record is outside the write surface, so `00-overview.md` gains the decision `d1-none-refusal-names-inbox-alone`.
- fix-consistency, LOW, the neither-word `bad-answer` widens a departure from D2's quoting of a receiving daemon's answer, unrecorded: fixed with my own change.
  `00-overview.md` gains the decision `d2-unusable-answer-is-a-code`, which covers the undecodable, foreign-hub-path and neither-word answers.

## Disputed

None.

## Deferred

None.
