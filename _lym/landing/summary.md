# Webster recovery gets a usable way forward; gate tests pass the audit

A batch that failed on fabric-reference findings now gets a recovery way forward the named verbs accept, and the fabric-reference audit no longer fails a batch whose fork ran the sanctioned `lym gate test` with a card argument.

Before, a fork that ran `lym gate test --card <id>` was flagged when the card ID ended in the weft suffix, because the bare weft-name rule read the ID as a sibling worktree.
`recover-batch` then refused with `needs_fresh` and advised `accept-audit`, which refused on the same findings, and `reset` was refused while the Webster session's own `run.lock` was held.
The session had no verb left and escalated.

Closes #877
Closes #878

## What changes

- `internal/fabricengine`: `ReferenceRule.MatchPath` blanks every `lym gate` segment in command position, up to the next shell separator, before its bare-word scan.
  A redirect ampersand (`2>&1`, `>&2`, `&>`) stays inside the segment, so an argument after it is still a gate argument.
- `internal/websterengine` `recover-batch`: when every uncheckable entry is a pathless fabric reference, the `needs_fresh` refusal runs the evidence check `accept-audit --batch` applies, mutating nothing.
  If the evidence holds, the way forward names `accept-audit --batch NN` then `recover-batch NN`.
  If it does not, the way forward is that refusal's reason and its own steps, ending in the re-run of both verbs, and it names no accept-audit route on its own.
  The refusal is still `ErrRecoveryNeedsFresh`.
- `internal/websterengine` `PlanReset`: the run-lock refusal for any target but `report-head` says the reset runs only once the run has stopped, that a Webster session inside the run ends it as stuck, and that the run's parent re-runs the reset once `lym webster status` shows no run in progress.
- `internal/planparser`: the card-id fabric-reference finding and `contracts/specs/loom-plan-spec.md` now say a command naming the card fails the audit unless it is a `lym gate` invocation.
- Package docs (`internal/fabricengine/doc.go`, `internal/websterengine/doc.go`, `internal/planparser/doc.go`) and tests are updated; new cases are rows in the existing tables.

## Design decisions

- One evidence function (`batchFabricReferenceEvidence`) serves both `accept-audit` and `recover-batch`, so the advice cannot drift from what `accept-audit` does.
  No guard changes: `recover-batch` still accepts in line only on the read-only basis.
- `reset --to batch-start` is not allowed inside the run.
  The Master stencil allows no reset but `report-head`, and the reset discards a batch the parent has not judged, so the refusal names who runs it and when instead.
- `IsReadOnlyCommand`'s allow-list is unchanged, and the finding's detail wording is unchanged.
- Only the bare-word scan skips a gate segment.
  The path rule and the spelling rule still run over it.

## Risk

A `lym gate` invocation that spells a sibling by bare name, such as a package pattern `other-weft/...` run from the hub, is no longer caught.
The path form `../other-weft/...` still is, as is a bare weft name in a segment after the gate segment.

## Not run by hand

- No end-to-end Webster run against a real failed batch; the refusal texts are covered by table tests only.
- `lym gate test --upto 2` on `main` before push, as the repository's push rule requires.
