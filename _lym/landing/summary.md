# Webster recovery gets a followable way forward; audit passes `lym gate`

A batch failed on fabric-reference findings could not be recovered from inside the run.
`recover-batch` pointed at `accept-audit`, which refused on the same findings, and `reset` was refused while the Webster session's own `run.lock` was held.
Separately, the fabric-reference audit failed a batch whose fork ran the sanctioned `lym gate test --card <card>`, because a card ID ending in the weft suffix was read as a sibling worktree name.

This fixes both: the audit no longer flags `lym gate` invocations, and each refusal on the recovery path names a route its verb accepts.

Closes #877
Closes #878

## Changes

- `internal/fabricengine/refscanner.go`: `ReferenceRule.MatchPath` blanks every `lym gate` segment in command position, up to the next shell separator, before the bare-word scan.
  A separator-prefixed weft path, the path rule and the spelling rule still run over the whole command, and a bare weft name in a segment after the gate segment still matches.
  A redirect's ampersand (`2>&1`, `>&2`, `&>`) stays inside the segment, so `lym gate test … 2>&1 | tail` is covered.
- `internal/websterengine/recoverbatch.go`: when every uncheckable entry is a pathless fabric reference, the needs-fresh way forward now runs the evidence check `accept-audit --batch` applies, mutating nothing.
  If it holds, the way forward names `accept-audit --batch NN` then `recover-batch NN`.
  If it refuses, the way forward is that refusal's reason and its own steps (typically `reset --to batch-start` first), ending in `accept-audit` then `recover-batch`.
  The error stays `ErrRecoveryNeedsFresh`.
  The evidence function is shared with `accept-audit` (`auditledger.go`), so the advice cannot drift from what that verb does.
- `internal/websterengine/reset.go`: `PlanReset`'s run-lock refusal now says the reset runs only once the run has stopped, that a Webster session inside the run ends it as stuck, and that the parent waits for `lym webster status` to show no run in progress, then re-runs the reset.
- `internal/planparser`, `contracts/specs/loom-plan-spec.md`, `internal/fabricengine/doc.go`, `internal/websterengine/doc.go`: docs and the card-ID finding's wording now say a `lym gate` invocation is exempt.
- `internal/fabriccli/hubmutation_integration_test.go`: the trace-sink assertion on the bypass push moves into its own serial test, since the sink is process-global and a parallel test's verb could repoint it between the push and the read.

## Design decisions

- Only bare words inside a `lym gate` segment are let through.
  A gate verb's arguments are gate keys, card IDs, tiers and package patterns resolved inside the worktree, never a sibling checkout.
- `reset --to batch-start` is not allowed inside the run.
  The Master stencil permits no reset but `report-head`, and the reset discards a batch the parent has not judged, so the refusal names the parent as the one who runs it.
- The audit's allow-list (`IsReadOnlyCommand`) and the finding's detail wording are unchanged.

## Risk

A `lym gate` invocation that spells a sibling by bare name, such as a package pattern `other-weft/...` run from the hub, is no longer caught.
The path form `../other-weft/...` still is.

## To check by hand

Not run against a live Webster run:

- A batch whose fork ran `lym gate test --wait --gate card --card <slug ending in -weft>`, chained after `git commit`, passes `record-batch`.
- `recover-batch` on a batch with non-read-only pathless fabric references prints the `reset --to batch-start` route and no `accept-audit` route of its own.
- `reset` under a held `run.lock` prints the parent-reruns-after-stop way forward.
