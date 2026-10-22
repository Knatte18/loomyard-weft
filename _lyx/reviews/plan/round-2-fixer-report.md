# Fixer report — plan, round 2

## Changes

- **F2 (LOW, consistency)**, fixed as proposed, plus the self-check's corrections, in `_lyx/plan/03-verifytree-pause-hold.md`.
  - **Intent:**
    - After the cancel sentence, a new sentence says every return after a hold removes the marker, whether `StatusDirty`, `StatusSkipped`, an error, a cancel or the start over's own result.
    - With a non-nil `site.Hold` only, Verify creates the verify directory and registers `defer os.Remove(p.Marker)` before its first `Wait`.
    - A nil-hold Verify keeps today's order and removes no marker on its dirty or skipped return.
    - Verify's doc comment gains the removal on every return after a hold.
  - **`TestVerify_PauseHold`:** these rows now assert that `ReadMarker` reports no live marker once Verify has returned:
    - the entry-hold pass row and the entry-hold dirty row;
    - the stop-then-pass, stop-then-dirty and stop-then-skipped rows.
  - **`doc.go` paragraph:** it now lists the marker's removal on every return after a hold.
  - **Premise checked:** today `Verify` registers the removal only after the running-marker write (`internal/verifytree/verify.go:323`), after the dirty and skip returns.

## Disputed

None.

## Deferred

- **F1 (MEDIUM, decision)**, deferred to the parent (`ly:orch`).
  - **Why no plan edit fixes it:** the review's premise holds, and the only sound change is outside this fixer's write surface, which is `_lyx/plan` plus this report.
    No plan edit removes either departure:
    - The declared `PATTERN-pause-holds-prompts` text renders as a 502-character entry line, over `pattern.MaxEntryLineChars` (360).
    - The overview is past `pattern.HeadroomBytes`, so its headroom lint refuses any growth.
  - **What the parent appends** to `_lyx/discussion/decision-record.md` is an `Added after Discussion` entry that records two things:
    - The `PATTERN.md` line of `PATTERN-pause-holds-prompts` is a shortened summary because the declared text is over the entry-line cap.
      The declared text is carried in full in `pattern/PATTERN-pause-holds-prompts.md`.
    - Card 8 cuts trailing clauses from eight entries and moves each, word for word, into its background file.
      The headroom limit licenses these cuts.
      The eight entries are `PATTERN-sparse-hub-base`, `-config-layers`, `-mutation-record`, `-friction-capture`, `-testkit`, `-hubforge-fixtures`, `-gogit-read-helper` and `-cwd-resolution`.
  - **After that:** once the entry exists, cards 8 and 9 and the overview's Shared Decisions bullet need no change.

## Self-check

- **fix-claims:** no findings.
- **fix-consistency, LOW:** the new sentence leaned on Verify's doc ("as Verify's doc says it does whatever happened"), but that clause covers only the run branch.
  Neither the Edit list's doc nor the `doc.go` paragraph carried the new removal.
  Fixed as proposed:
  - I dropped the appeal to the doc.
  - The card now says Verify's doc comment gains the post-hold removal.
  - The `doc.go` paragraph now lists it.
- **fix-proposal, LOW:** the early directory creation and `defer` were not scoped to a non-nil hold.
  Moved to the top of `Verify`, the `defer` would let a nil-hold dirty or skipped return delete another run's marker, which breaks "a nil hold runs exactly as today".
  Fixed as proposed: the card scopes both to a non-nil `site.Hold` and keeps the nil-hold order.
- **fix-affected-set, LOW:** the entry-hold pass row and the stop-then-pass row return the start over's result after a hold, but they lacked the no-live-marker assertion.
  Fixed as proposed: both rows now carry it.
