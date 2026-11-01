```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I checked whether the card 2, 5 and 9 changes agree with the sibling cards and cross-references.

no findings

- The overview's shared decision "only reedengine's `placeButtons` decodes it, at render, so a bad button costs only itself (D2)" agrees with card 2's new opening sentence. The open maps are what make that claim hold.
- Card 3's `placeButtons` rows match card 2's new load behaviour, so nothing there needs to change. They skip a group written as a mapping, and they return nothing for a node that lacks the selected group.
- Card 6's `ButtonGroupMappingPasses` row loads `buttons: {task: {label: x}}`, which agrees with card 2's row saying that mapping loads under the open maps.
- Card 2's desktop bullets still leave desktop without open maps and refuse mapping groups at the machine layer. This matches the review's statement that desktop is unaffected.
- Card 2's Edit/Extends move of `docs/overview.md`, card 9's move of `internal/toolfloor/doc.go`, and card 5's added header sentence have no twin elsewhere.

Tool calls:
- Read `_lym/plan/00-overview.md` lines 15–40 and grepped cards 3 and 6 for button shape and load wording. My context did not show those sibling cards.
