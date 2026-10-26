# Decision record — weft-suffix-card-slug-audit

## Goal

A card slug is never read as a weft sibling worktree by the implementer's fabric-reference audit, so a fork that runs `lyx gate test --card <card>` for its own card is never failed for a fabric reference it did not make.

## Scope

- The plan gate refuses a card whose card id the fabric-reference path rule matches, under the existing `card-fabric-reference` check in `internal/planparser/fabricref.go`, with a way forward that names a rename of the card.
- `contracts/specs/loom-plan-spec.md`'s `card-fabric-reference` entry and the `internal/planparser` package doc describe the new hit.
- The implementer audit in `internal/fabricengine/refscanner.go` stays as it is.

## Decisions

### Refuse the slug at the plan gate, not in the audit

Decision: `CheckCardFabricReference` runs the matcher's path rule over each card's card id (`N-<slug>`) and reports a finding naming the card, the matched text and the way forward: rename the card so its slug no longer ends in the matched suffix.
Rationale: a card id appears in many commands besides `--card` (state paths, commit messages, grep patterns), and the audit reads every one of them; refusing the name once at the plan gate covers every spelling, while the audit keeps every guard it has.

### Same check ID

Decision: the new hit is a `card-fabric-reference` finding, not a new check ID.
Rationale: it is the same rule with the same matcher, and the plan writer's fix lives in the same place; a new ID would add a row to the spec's ordered check list for no gain.

### planparser keeps its naming-free seam

Decision: the check reaches the suffix only through the `FabricReferenceMatcher` it already receives, so `internal/planparser` imports no weft naming.
Rationale: the seam's contract is that planparser imports neither the fabric side nor its naming.

## Constraints

- `PATTERN-sole-parsers`: planparser stays the plan's sole parser; the check reads the parsed `Plan` only.
- `PATTERN-refusal-way-forward`: the finding names the way forward and a test reaches it.
- `PATTERN-test-speed`: the new behavior is a row in the existing tier0 `TestCheckCardFabricReference`.

## Auto-mode assumptions

- A plan already approved before this change with a `-weft` card slug is rare enough that a rework refusing it is acceptable.

## Open risks

- A rework of a plan whose done card carries a `-weft` slug is refused, since the check takes no done set; the operator renames the card or accepts the batch audit by hand.

## Acceptance criteria

- A plan with a card slug ending in `-weft` fails `card-fabric-reference`, the finding naming the card id and a rename as the way forward.
- A card slug that only contains `weft` without ending in the suffix, such as `weft-prime-marker`, passes.
- Existing `card-fabric-reference` rows keep their outcomes.
