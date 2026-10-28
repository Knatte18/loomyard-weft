# Decision record — webster-recovery

## Goal

A batch failed on fabric-reference findings gets a way forward that its named verbs accept, and the fabric-reference audit stops failing a batch whose fork ran the sanctioned `lym gate test` with a card argument.

## Scope

- In: `internal/fabricengine/refscanner.go`'s bare weft-name rule, `internal/websterengine`'s recover-batch needs-fresh way forward and accept-audit evidence check, and PlanReset's run-lock refusal text; their tests and the package docs that describe them.
- Out: the Master stencil's ladder, which already ends the run stuck on `needs_fresh` and `audit_not_acceptable`; the finding's detail wording; IsReadOnlyCommand's allow-list.

## Decisions

### D1: A `lym gate` segment's words are never a bare weft sibling name

**Decision:** `ReferenceRule.MatchPath` blanks every `lym gate` segment in command position, up to the next shell separator, before the bare-word scan; the path rule and the spelling rule still run over it unchanged.
**Rationale:** the incident's card ID `6-clone-records-marker-on-prime-weft` ends in the weft suffix, and the bare rule read it as a sibling worktree.
A gate verb's arguments are gate keys, card IDs, tiers and package patterns, resolved inside the worktree, never a sibling checkout.
Bound: only a bare word inside a `lym gate` segment is let through; a separator-prefixed weft path, the worktree's exact sibling path and every other command keep matching.

### D2: recover-batch's needs-fresh way forward runs accept-audit's own evidence check

**Decision:** when every uncheckable entry is a pathless fabric reference, recover-batch evaluates the evidence `accept-audit --batch` applies, mutating nothing.
When it holds, the way forward names `accept-audit --batch NN` then `recover-batch NN`; when it refuses, the way forward is that refusal's reason and its own steps, ending in `accept-audit --batch NN` then `recover-batch NN`.
The refusal stays `ErrRecoveryNeedsFresh`.
**Rationale:** the old text offered `accept-audit` on a basis recover-batch had already ruled out, since the read-only basis is accepted in line before the refusal.
One evidence function serves both verbs, so the advice cannot drift from what accept-audit does.
No guard changes: recover-batch still accepts in line only on the read-only basis.

### D3: reset's run-lock refusal names who runs the reset and when

**Decision:** PlanReset's refusal for a target other than `report-head` while `run.lock` is held names the way forward as ending the run first, with Master's run stopping as stuck, and the parent re-running the reset once `lym webster status` shows no run in progress.
**Rationale:** the Webster session that holds the lock cannot run the reset, so the advice to wait gave it nothing to act on.
Allowing `batch-start` inside the run is left out: the Master stencil allows it no reset but `report-head`, and the reset discards a batch the parent has not judged.

## Constraints

- `PATTERN-refusal-way-forward`: each changed refusal names its way forward and a test reaches it.
- `PATTERN-test-speed`: new cases are rows in the existing table tests, at the existing tiers.
- Docs land in the same commit: `internal/websterengine/doc.go` and the scanner's doc comments.

## Auto-mode assumptions

- `lyx` in the board entry is another hub's build of `lym`; the fix targets the `lym` spelling the code matches.
- The fork's command was flagged by the bare weft-name rule on its `--card` value, as the archived refusal note's commands show; no other rule matched them.

## Open risks

- A `lym gate` invocation that spells a sibling by bare name, such as a package pattern `other-weft/...` run from the hub, is no longer caught; the path form `../other-weft/...` still is.

## Acceptance criteria

- The scanner does not match `lym gate test --wait --gate card --card 6-clone-records-marker-on-prime-weft 2>&1 | tail -40`, nor the same command chained after `git commit -m "…" &&`.
- The scanner still matches a `lym gate` segment holding a separator-prefixed weft path, and a bare weft name in a segment after the gate segment.
- recover-batch on a batch whose pathless fabric references are not read-only, at HEAD past the start, names `reset --to batch-start --batch NN`, then `accept-audit --batch NN` and `recover-batch NN`, and names no accept-audit route on its own.
- recover-batch on such a batch at HEAD equal to its start on a clean tree names `accept-audit --batch NN` then `recover-batch NN`.
- recover-batch on such a batch with no recorded start names the reset-to-start route only.
- accept-audit's own refusals keep naming `accept-audit --batch NN` for the re-run.
- PlanReset under a held run lock names the parent re-running the reset after the run stops.
