# Plan review round 1 — fixer report

Every finding's premise held against the code; all fourteen are fixed.
`lyx loom validate-plan` and `lyx loom validate-plan --lens-review` both pass after the changes.

## Changes

- **F1** (MEDIUM) — fixed as proposed, the carry-D4 branch.
  Card 8 adds `internal/configsync#ReconcileHubWideAt` to its Edit list and an Intent bullet: on a dry run the module loop reports no seed additions for a module whose absent board file the same pass's key moves would write, for every move target.
  The dry-run row now asserts, through a new `noAdded` harness field, that the `lenses` result carries no `Added`.
  The D4 bullet is dropped from `00-overview.md`'s `## Shared Decisions` and from card 8, and card 8's ImpactSummary names the dry-run change.
  The route-through-parent branch was not taken: this fixer sends no message to another session about the review.
- **F2** (LOW) — fixed as proposed: card 5 says `TestRunCmd`'s engine-without-forks subtest builds a `lenses.Config` that defines `standard` and its lenses.
- **F3** (LOW) — fixed as proposed: card 5's doc sweep covers burlerengine `doc.go`'s "# The fixer fan" section; card 6's `docs/overview.md` sentence covers the eight fan keys of the loom config paragraph.
- **F4** (LOW) — fixed as proposed: card 3's `ResolveFan` refusals carry the `lenses: ` prefix, and `TestResolveFan`'s refusal rows assert it.
- **F5** (LOW) — fixed as proposed: card 2 targets `internal/configengine/doc.go` and gains a per-name maps paragraph sentence.
- **F6** (LOW) — fixed as proposed: card 1's per-name value shape follows the kind of the template's entries at P (string scalar under `lenses`, flow list through `replaceSequence` under `fans`, a non-list refused).
  The rows run over a template with a scalar-entry map `names` and a list-entry map `lists`; `TestSetValues` gains the `[`-prose-as-string row and `TestSetValues_Refusals` the `lists.<name>=plain` row.
- **F7** (LOW) — fixed as proposed: card 3's `CheckLoad` bullet and its `doc.go` bullet say several failing fans take every fix in one `--set` or one `--edit`.
- **F8** (LOW) — fixed as proposed, with the scope tightened after the self-check: a pair under P whose existing value at P is neither a mapping nor null is refused naming P, its shape and the way forward; `TestSetValues_Refusals` and `TestSetSparse` each gain a row.
- **F9** (LOW) — fixed as proposed: card 1's `Reconcile` non-mapping refusal ends with `rewrite P as a map of name to value`, and the `wantErr` row asserts it.
- **F10** (LOW) — fixed as proposed: card 8 names the next step for a `burler` run-settings object, adds `internal/configreg#PrepareRunSettings` to its Edit list to carry the way forward in the refusal text, and `TestPrepareRunSettings`'s two rows assert it.
  The `lenses.local.yaml` bound names deleting the local file as the way forward.
- **F11** (LOW) — fixed as proposed: `TestSetValues` gains the null-at-`names` row.
- **F12** (LOW) — fixed as proposed: the `TestPrintResolved` subtest and its Edit entry are dropped, and card 3 says the criterion rests on the registration rows and the existing annotation subtest.
- **F13** (NIT) — fixed as proposed: card 6 says a temp dir holding no `_lyx/`, which degrades to the template.
- **F14** (NIT) — fixed as proposed: card 4 targets `internal/hubreconcile#logResult`, which logs `keptExtra` beside `keptTarget`.
  The plan validates with the added target.

## Self-check

- fix-consistency — LOW — `lens-review.md` lines 8 and 9 stale against F1 and F6 — fixed as proposed: both entries now record the round-1 change.
- fix-consistency — LOW — card 4's ImpactSummary missed the `logResult` change — fixed as proposed.
- fix-consistency — LOW — F8's refusal had no `TestSetSparse` row — fixed as proposed: `TestSetSparse` gains the row.
- fix-claims — no findings.
- fix-proposal — LOW — F8's refusal was not tied to a pair under P — fixed as proposed: the sentence and both rows now name a pair under `names`.
- fix-affected-set — MEDIUM — `TestResolveFan`'s fallback subtests would fail when moved unchanged — fixed as proposed: card 3 drops the two fallback subtests, naming the tests that own their criteria, and renames the undefined-in-both subtest.
- fix-affected-set — MEDIUM — `docs/overview.md`'s Discussion-Write and Plan-Write lens-fork sentences lacked a carrier — fixed as proposed: card 6's sentence names both.
- fix-affected-set — LOW — `resolveRecordedFan`'s doc comment still names `burler.yaml` — fixed as proposed: card 5 retargets it.

## Disputed

None.

## Deferred

None.
