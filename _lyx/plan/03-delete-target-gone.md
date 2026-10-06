# Card 3 — delete-target-gone

**Edit:**
- `internal/planglyph/planglyph.go`
- `internal/planglyph/resolve.go`
- `internal/planglyph/repo.go`
- `internal/planglyph/doc.go`
- `internal/planglyph/planglyph_test.go`
- `internal/planparser/validate.go`
- `internal/websterengine/beginbatch.go`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/cli_test.go`

**Intent:**
Implement decision D3: once a run has begun changing the tree, a **Delete:** target of an unbegun card that no longer exists stops blocking `begin-batch` (`webster-delete-already-gone`) and becomes an informational advisory.

In `planglyph.ValidateDispatch`, when `completed` is non-empty, a **Delete:** group target of a pending card that is absent — a path or self glyph `path-missing` reports, or a glyph the resolve pass answers `not_found` — is reported as one informational finding, check ID `delete-target-gone`, attributed to that card, naming the target as already deleted; that card's blocking `path-missing` or `glyph-not-found` finding for that target is not reported.
Every other finding for the same target stays: a pending card that names it under `Edit:`, `Uses:` or a Rename old side still gets its blocking finding.
With `completed` empty `ValidateDispatch` keeps returning exactly `ValidateFormat`'s answer, and `ValidateFormat`, `Validate` and `ValidateRework` never downgrade, so the Plan-Gate, the Rework-Plan-Gate, run entry on a fresh run and `lyx webster validate` before any batch is begun keep refusing a missing Delete target as a plan typo.
To drop exactly the right pure finding, `planparser.ValidationError` gains a `Ref` field that `checkPathMissing` fills with the raw ref it reports, and `planglyph.Finding` carries it through `fromValidationError`; `statusFindings` fills `Ref` on the findings it emits.
How `resolvePass` learns it is the dispatch form (a parameter, or a post-pass filter in `ValidateDispatch`) is the implementer's call, as long as the whole-plan entry points cannot reach the downgrade.

Bound, stated in `ValidateDispatch`'s doc comment: this lets through a Delete target missing for any reason, including one a card removed by editing the wrong file or one only a mid-run plan edit added; the Plan-Gate and `lyx webster validate` saw the target exist before the run started, so only a target the run removed or a mid-run edit added reaches it, and the advisory names the target and the card.
`record-batch`'s `delete-not-done` does not bound it, since it reads a not-found target as the wanted state.

`BeginBatch` already returns informational findings in `BeginResult.Advisories`; update its comment block on the dispatch re-resolution to say an already-deleted Delete target arrives there.
Add `delete-target-gone` (informational) to `internal/planglyph/doc.go`'s check list, and to the `loom-plan-spec.md` paragraph card 2 added only if that paragraph enumerates resolve-backed IDs.

Tests:
in `planglyph_test.go`, `ValidateDispatch` with one completed card and a pending card whose Delete target (once as a file path, once as a member glyph) is absent returns `delete-target-gone` as informational and no blocking finding for it; a pending card that `Uses:` the same absent target still gets its blocking finding; with no completed cards the same plan returns the blocking finding.
In `internal/webstercli/verbs_test.go`, drive `begin-batch` through the cobra command on the verbs fixture: batch 1 removes a file that card 2 also lists under **Delete:**; batch 1 recorded, `begin-batch 2` succeeds and its envelope's advisories name the target as already deleted.
In `internal/webstercli/cli_test.go`, `lyx webster validate` over the same plan with no run state exits 1 with the missing Delete target as a blocking finding.

**ImpactSummary:** ValidateDispatch gains a scoped downgrade for absent Delete targets of pending cards once a batch is begun; findings gain a Ref field; the whole-plan entry points are unchanged.
