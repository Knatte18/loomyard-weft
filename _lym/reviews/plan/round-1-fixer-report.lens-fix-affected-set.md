```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I reran the set searches behind F1 and F4/F9 and checked that the fix covers every member.

Unchecked findings: F2, F3, F6 and F7 have no search-based set to rerun beyond what the fixer already did. F5's set is the skill names, which the fix collects from the built specs.

**MEDIUM: the fanned assistant stencil is still outside the catalog**
- Changed text: card 6 Intent, "`RunOpts` whose `Assist` names a stand-in model, so the assistant seat … is in the catalog."
- Evidence: `internal/burlerengine/assist.go:142-144` switches the seat to `assistFannedReviewerStencil` when `opts.AssistFan` resolves lenses.
- The fixer's `Build` names no `AssistFan`, so the catalog only ever builds the solo assistant stencil.
- `burler-template-assist-reviewer-fanned` is a second assistant variant, and neither the catalog nor the scan reaches it. The scan only sees the `&seatengine.Seat{` literal in `assistSeat`, which is now listed, so nothing fails on the gap.

**MEDIUM: the fanned burler review and fix seats are not built, against the F4 ruling**
- Changed text: card 6's burler `Build`, "a `Profile` with stand-in review and fixer-report paths". Also card 6's TestSeats sentence "For every `Seat`, a seat Codex cannot start included".
- Evidence: the ruling names "fanned burler review and fix" as compared.
- `review.go:87` sets `ForkSubagents: p.ClusterFan != ""` and `fix.go:97` sets `ForkSubagents: p.FixFan != ""`.
- The stand-in `Profile` sets neither `ClusterFan` nor `FixFan`, so the catalog only holds the unfanned variants.
- A fanned seat's manifest differs: card 3 adds `webster-fork-deny` for a `ForkSubagents` spec. The fanned seats the ruling names are therefore never compared.

Covered with no finding:
- **Sites that change a table after its literal:** `discussiontable.go:142` and `partsplan.go:145` append inside their listed builders, and `assist.go:157-159` is `withAssistant`, reached through `probeTable`.
- **Specs with forks or requirements:**
  - `websterengine/runlevel.go:768-770` (Run) and `loomcli/driverspec.go:62-63` are both named in card 7.
  - `orchcli/start.go:224` is a skipped place.
  - `seatengine/prompt.go:231` is `seatSpec`.

Tool calls:
1. Grep for `.Stencil =` and `Seats` reassignment or append in production code. My context had only `assist.go`'s mutations, not the whole repository.
2. Grep for `ForkSubagents: true` and `Requires:` in production code, to rerun the F4/F9 set search across the repository.
