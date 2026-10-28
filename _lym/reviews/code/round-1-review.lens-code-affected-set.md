```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, meaning what the diff's changes force outside the lines it touches. I found one NIT and no other finding; each change's affected set is listed below.

**Per change, the set found**
- **Orch walk moved to `orchroute`, `orchcli` delegating (cards 1, 4):** covered.
  - Remaining `orchcli` callers (`beamcli`, `dobbydeps`, `loomcli/decision.go`, `tendercli/deps.go`) still compile through the delegations.
  - Loom's hand-back and landing notices (`loomcli/landingdeps.go:204`, `drivewiring.go:243`, `wiring.go:587`) keep `NotifyParent`, as D3 says.
- **Notifier wirings (D3):** covered.
  - `SetNotifier` has exactly three non-test call sites: burlercli, loomcli and webstercli.
  - All three now call `NotifyAsker`.
- **Removal of `lym shuttle send` and `lym shuttle interrupt` (D8):** covered, except the NIT below.
  - No live reference is left in `internal`, `docs/overview.md`, `pattern/`, `PATTERN.md`, `contracts` or `tools`.
  - The remaining hits are `cmd/lym/removedverbs_test.go`, which pins their absence, and the dated `docs/research/` reports, which are history.
  - The `ErrNoRunningTurn` text and the `run_test.go` rows now name `lym creel post`.
  - `docs/overview.md:414` now says a line typed into a pane goes through creel.
- **`RequirementPeerMessaging` removal (D7):** covered.
  - No `PeerMessaging` identifier is left anywhere.
  - `peer-messaging` appears only in `requires_test.go`, as the unknown-name rows, and the "known requirements" text reads `forks, background-wake`.
- **`SendMessage` in stencils and Go text (D2, D4, D5, D13):** covered.
  - The only stencil naming it is the webster Merriam fork line, which the scan allowlists (`agentmessaging_test.go:29`).
  - The `internal/orchcli/smoke_cycle_test.go` and `claudeengine/testdata/corpus/*.jsonl` mentions are outside the scan's scope.
  - The scan covers stencils, `.md` fixtures under `internal/**/testdata` and non-test Go literals.
  - The `.jsonl` corpus files are transcripts, not prompt fixtures.
- **Docs, PATTERN and commits:** covered.
  - `PATTERN.md` carries the `PATTERN-agent-messaging` line with the declared text.
  - `pattern/PATTERN-pane-text-injection.md` and `PATTERN-daemon-wakeups.md` are updated.
  - `creelengine/doc.go` no longer has an "Unbuilt" or stage 3 section.
  - `orchengine/doc.go`, `seatengine/doc.go`, `creelcli/doc.go`, `reportcli/doc.go`, `reportengine/doc.go` and `orchroute/doc.go` are touched.
  - No commit carries a `Pattern-Departure:` trailer, so none needs judging.
- **Message line, `MessageLine` and `LineFits` (D9):** covered. `LineFits` and `validateLine` now measure the widest marked form through `widestLine`.
- **`appendPostedLocked` signature change:** I did not check every caller. The `record`-argument form is used at the `storePost`, `Reply`, `postChaseLocked` and `noticeUnansweredLocked` sites I saw.

**Finding**
- **NIT, consistency.** `internal/creelengine/doc.go:290` says "where the removed shuttle send refused it", which names the removed verb in a live doc. D8 asks docs to name the creel verb instead. Fix: reword to "where a refused pause-time send used to refuse it", or drop the clause.

**Tool calls**
1. A `grep` for shuttle send and interrupt, peer-messaging and `SendMessage` references and for `SetNotifier` and `NotifyParent` call sites. My context held only diff excerpts, not whole-tree search results.
2. A read of `smoke_cycle_test.go` and the `SendMessage` hits, plus a grep for `ErrNoRunningTurn` and `orchcli` consumers and the wiring diffs. I needed to confirm the callers still resolve and that the three notifier wirings changed.
3. A grep of the scan test's scope and allowlist, a search for `SendMessage` in non-Go text, and a search for the "Unbuilt" section. I needed to confirm what the scan covers and that the doc section was replaced.
