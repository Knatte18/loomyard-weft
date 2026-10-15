# Support log — webster-pack-layout-switch

## Interview

### Turn 1 — exploration (autonomous; no operator, no advisors)

Read the board entry, `glyph-use-audit` (the audit and its results), `lighter-test-rules` and `weaver`.
Read `1e39afc10`, `d8d1c0956` and `793c6a8ee`, and diffed the pack fragments across `113593990`, `793c6a8ee` and `1e39afc10^`.
Findings:

- The pack-related fragments at `1e39afc10^` equal `793c6a8ee`'s; between them only the body's unrelated lines and the master template changed.
- At `113593990`, FRESH-READ had no one-turn instruction; step 1 read card by card.
  The board's description of the measured layout and the audit's "Read-all-first (`793c6a8ee`)" result both match `1e39afc10^`, so the entry's commit cite names the predecessor.
- `configengine` fills template keys the merged layers lack, so a new key needs no hub file edit.
- The recovery prompt shares the implementer body and lists `kickstart_pack` as optional.

### Turn 2 — design picks (auto)

Settled Q1–Q6 in the ledger by own judgment, leading with the recommendation each time.

### Turn 3 — lens review by forks

All six forks returned a report.

- goal-scope MEDIUM (D2 overrides the board's `113593990` cite as a bare auto-pick): fixed; Auto-mode assumptions asks the parent to confirm and notes the cite may want correcting.
- goal-scope LOW (empty-pack rule departs from `1e39afc10^`): rejected; already named in Auto-mode assumptions, and no measured fork reads an empty pack.
- goal-scope LOW (inline is not a byte copy): rejected; already an open risk.
- premises LOW (audit's body had the `deviations` field): fixed; folded into the open risk.
- premises LOW (`pack_layout` placement breaks log pins): fixed; D5 puts it after `unresolved`.
- premises LOW, leanness LOW (Notes repeat D3 and D2): fixed; Notes trimmed.
- scenarios MEDIUM (no criterion for a flip on a re-run of the same batch): fixed; the flip criterion covers both directions on a re-run.
- scenarios LOW (how a non-bool value is refused): fixed; criterion names `--set`'s refusal.
- scenarios LOW (no prompt carries both sections): rejected; implied by the two byte-identity criteria.
- scenarios LOW (degraded pack), no-finding (concurrency): no change needed.
- leanness LOW (D2 rationale is narrative): fixed; cut to one line, evidence stays in Turn 1.
- leanness LOW (Auto-mode assumptions restate decisions): fixed; bullets point at D1, D2, D4, D5.
- leanness LOW (Open risks duplicate Scope and D2): fixed; stale-file bullet cut, the other kept for the measurement confounder.
- leanness LOW, pattern-fit LOW (padding Constraints): fixed; `PATTERN-tolerant-records`, `PATTERN-log-levels`, the test-budget copy and the review-budget advice cut.
- leanness LOW (Scope "In" restates decisions): fixed; one line.
- leanness LOW (acceptance criterion names tests): fixed; rephrased as behavior.
- board-fit LOW (board cite conflict): merged with goal-scope MEDIUM.
- board-fit LOW (audit scripts' key match): fixed through D5's placement.
- board-fit LOW (heavy wording pins vs `lighter-test-rules`): rejected; the text-equality criterion is the layout's contract, and that note is undecided.
- pattern-fit MEDIUM (inline fragments' drift rule unstated): fixed; D3 makes them frozen baseline copies.
- pattern-fit LOW (marker pin unnamed): rejected; Notes already name `markers_test.go`.
- pattern-fit LOW (`ImplementerBodyTemplate` reader): fixed; D3 names it.

## Rejected alternatives

- **Inline layout as at `113593990`** (per-card step-1 reads): the entry asks for the layout the audit measured, which was read-all-first; restoring per-card reads would compare against a layout no audit result describes.
- **An enum key `kickstart_pack_layout: file|inline`**: more open-ended than the two layouts the entry names; a bool mirrors `kickstart_pack` and the entry's on/off wording.
- **Moving the single `kickstart_pack` marker by layout through Go string splicing**: puts prompt structure into Go, against `PATTERN-stencil-ownership`; a second optional marker keeps the structure in the body stencil.
- **Parameterising the existing pack fragments with markers for the differing words**: makes both layouts' text harder to read and diff against `1e39afc10^`; separate files keep each layout whole.
- **Logging `pack_layout` only on `on` lines**: a key that comes and goes is harder to grep than one with `none`.
- **Mirroring later file-layout fragment edits into the inline fragments**: the inline layout is a measurement baseline; keeping it in step would erase the difference being measured.
- **Removing a stale `<batch>-pack.md` when inline renders**: the pack-off path leaves the same stale file today, the inline prompt never names it, and deletion adds a destructive step for no measurement gain.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): Which commit's layout is "off"? — `1e39afc10^`, the read-all-first layout the audit measured; the entry's `113593990` cite is its predecessor (D2).
- Q2 (auto-pick, resolved): Key shape and name? — bool `kickstart_pack_file`, default `true` (D1).
- Q3 (auto-pick, resolved): How does the body place the inline section? — a second optional marker `kickstart_pack_inline` at `1e39afc10^`'s position, with three new inline fragments (D3).
- Q4 (auto-pick, resolved): Does the empty-pack rule apply inline? — yes, it is decided before the layout (D4).
- Q5 (auto-pick, resolved): Log key? — `pack_layout` with `file`, `inline` or `none` on every line (D5).
- Q6 (auto-pick, resolved): Edit hub config files? — no; the template default fills a missing key (D1).
